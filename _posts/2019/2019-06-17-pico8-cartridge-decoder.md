---
layout: post
title:	"PNG cartridge decoder: a tale of unneeded optimisation"
date:	2019-06-17 12:10:00
categories:
    - blog
    - pico8
tags:
    - coding
    - rust
    - pico8
    - assembly
    - optimisation
    - bit-twiddling
---

Remember that
[Donald Knuth quote](https://en.wikiquote.org/wiki/Donald_Knuth#Computer_Programming_as_an_Art_(1974))
that goes ***"premature optimisation is the root of all evil"***? Yeah, this is
yet another post about that.

# Cartridge format

PICO-8 cartridges are stored in PNG format, which is perturbing at first
since clicking on the *Download cart* button on the
[Lexaloffle BBS](https://www.lexaloffle.com/bbs/?mode=carts) redirects you to
a PNG image instead, but that image actually contains the cartridge data itself!

<table style="margin-bottom: 10px;">
  <tr>
    <td>
      <a href="https://www.lexaloffle.com/bbs/?pid=62911#p">
        <img src="https://www.lexaloffle.com/bbs/cposts/pi/picotunes2-0.p8.png" style="width:160px; height:205px;" alt="Picotunes II"/>
      </a>
    </td>
    <td>
      <a href="https://www.lexaloffle.com/bbs/?pid=64346#p">
        <img src="https://www.lexaloffle.com/bbs/cposts/fu/fuz_v1-1.p8.png" style="width:160px; height:205px;" alt="FUZ"/>
      </a>
    </td>
    <td>
      <a href="https://www.lexaloffle.com/bbs/?pid=60313#p">
        <img src="https://www.lexaloffle.com/bbs/cposts/gi/gifts_on_venus-0.p8.png" style="width:160px; height:205px;" alt="Gift on Venus"/>
      </a>
    </td>
    <td>
      <a href="https://www.lexaloffle.com/bbs/?pid=54347#p">
        <img src="https://www.lexaloffle.com/bbs/cposts/ge/getoutxmas-0.p8.png" style="width:160px; height:205px;" alt="Get Out Of This Dungeon"/>
      </a>
    </td>

  </tr>
  <tr>
    <td colspan="100%">
      <p style="margin: 10px; text-align: justify;"><i>
        Some nice cartridges found online. If you take a closer look, you'll
        notice the gray area is actually not a flat color but show a lot of
        artifacts, as in a JPEG image.
      </i></p>
    </td>
  </tr>
</table>

In order to decode a cartridge, the raw
[*RGBA*](https://en.wikipedia.org/wiki/RGBA_color_space) bitmap must be decoded
from the PNG first. The two PNG decoders I could find for Rust were
[`png`](https://docs.rs/png) and [`lodepng`](https://docs.rs/lodepng);
I found the latter easier to use, as it decoded the complete image at once,
which is okay since cartridge images are fairly small (160x205 pixels).

# Decoding the naive way

The PICO-8 cartridge is encoded in the bitmap using a
[**steganographic** encoding](https://en.wikipedia.org/wiki/Steganography): each
pixel of the cartridge actually contains a byte of cartridge data, hidden in the
two least significant bits of each channel.

       red   0bxxxxxxRR
       blue  0bxxxxxxBB
       green 0bxxxxxxGG
       alpha 0bxxxxxxAA
       ----------------
       byte  0bAARRGGBB

The naive implementation, as found in `picotool`, is to use a bitmask and a
shift to read each channel (no worries about the constant multiplication,
they are evaluated by the compiler and increase the code legibility):

```rust
for offset in 0..0x8000 {
    let pixel: RGBA = image.buffer[offset];
    cart[offset] |= ((pixel.b & 3) << (0 * 2));
    cart[offset] |= ((pixel.g & 3) << (1 * 2));
    cart[offset] |= ((pixel.r & 3) << (2 * 2));
    cart[offset] |= ((pixel.a & 3) << (3 * 2));
}
```

We notice however that by treating a pixel as a single 32-bits integer, we could
reduce the total number of atomic operations; for that, we can only bitmask once,
and then use shifts to put the decoded bits at the right location in the final
byte.

# Masking the pixel only once

After decoding PNG data with `lodepng`, we're left with a
[`Bitmap<RGBA>`](https://docs.rs/lodepng/2.4.2/lodepng/struct.Bitmap.html)
(that is actually a `Vec<RGBA>` with the image dimensions stored alongside).
Fortunately, the [`RGBA`](https://docs.rs/rgb/0.8.13/rgb/struct.RGBA.html) struct
[is declared as `#[repr(C)]`](https://docs.rs/rgb/0.8.13/src/rgb/lib.rs.html#71-90)
so we should be able to transmute that into a slice of 32-bit integers.
*This is not 100% safe since we could face alignment errors, but that should work
most of the time.*

Here is the algorithm (with `0` bits shown as `_` for legibility purposes) on
a little-endian platform:

       |-------------pixel--------------|              |----------command---------|

       0bxxxxxxAAxxxxxxBBxxxxxxGGxxxxxxRR              rotateleft pixel, 8
    -> 0bxxxxxxBBxxxxxxGGxxxxxxRRxxxxxxAA              byteswap   pixel
    -> 0bxxxxxxAAxxxxxxRRxxxxxxGGxxxxxxBB              and        pixel, 0x03030303
    -> 0b______AA______RR______GG______BB                   

We only have to perform a single bitwise `and` to mask all our channels. Then
we can collect each byte using a single `or` and `shift right`:

       |-------------pixel--------------|  |--byte--|  |----------command---------|
       0b______AA______RR______GG______BB  0b________  or           byte,  pixel
    -> 0b______AA______RR______GG______BB  0b______BB  shiftright   pixel, 6
    -> 0b____________AA______RR______GG__  0b______BB  or           byte,  pixel
    -> 0b____________AA______RR______GG__  0b____GGBB  shiftright   pixel, 6
    -> 0b__________________AA______RR____  0b____GGBB  or           byte,  pixel
    -> 0b__________________________RR____  0b__RRGGBB  shiftright   pixel, 6
    -> 0b________________________AA______  0b__RRGGBB  or           byte,  pixel
    -> 0b________________________AA______  0bAARRGGBB


The following is the Rust implementation:

```rust
let bitmap = image.buffer.as_slice().as_ptr() as *const u32;
for offset in 0..0x8000 {
    let mut pixel = (*bitmap.add(offset)).rotate_left(8).to_be().bitand(0x03030303);
    let mut res = pixel as u8;      
    pixel >>= 6; res |= pixel as u8;
    pixel >>= 6; res |= pixel as u8;
    pixel >>= 6; res |= pixel as u8;
    cart[offset] = res;
}
```

# The Bit Manipulation Instruction set

Although I tried at first to write inline assembly with the `asm!` macro, it turns
out this is not really a good idea because the compiler can't optimise your code
anymore, and it was made clear by benchmarks this was a *really* bad idea.

Rust however recently stabilized some architecture intrinsincs in the `std::arch`
module, so I started exploring that option, and in particular some additional
instruction sets. It turns out the **Bit Manipulation Instruction Set** (short. BMI2)
has a `PEXT` instruction that does exactly what we need: given an integer and
a bitfield, `PEXT` will pack all bits selected by that bitfield.

    pixel    0bxxxxxxAAxxxxxxRRxxxxxxGGxxxxxxBB
    bitfield 0b00000011000000110000001100000011
    -------------------------------------------
             0b000000000000000000000000AARRGGBB

This call lets us extract out the encoded byte from our pixel data with a single
instruction. *Neat!* On `x86_64` systems, it's even possible to call it on a
64-bit integer, which lets us process two pixels at once.

Here are the benchmarks without BMI2 support:
```rust
test naive            ... bench:      42,295 ns/iter (+/- 1,435)
test rotate32         ... bench:      19,726 ns/iter (+/- 2,989)
```
and here are the results with BMI2 enabled:
```rust
test naive            ... bench:      47,357 ns/iter (+/- 1,333)
test rotate32         ... bench:      20,361 ns/iter (+/- 602)
test pext32_bmi2      ... bench:      20,081 ns/iter (+/- 646)
test pext64_bmi2      ... bench:      14,165 ns/iter (+/- 636)
```

# Parallelizing even more

So, as it seems, the boost in performance did not come from `PEXT` alone,
but with the fact we are processing more than one pixel at a time. Luckily, this
is what the Intel **Single Instruction Multiple Data** (short. SIMD) and
**Advanced Vector Extensions** (short. AVX) are for. With AVX, we can process
256 bits (8 pixels) at a time.

Using the [`_mm256_shuffle_epi8`](https://doc.rust-lang.org/nightly/core/arch/x86_64/fn._mm256_shuffle_epi8.html)
with the appropriate mask, we can shuffle our bytes to have each pixel as `ARGB`,
and then use several `PEXT` calls to pack our bytes.

Here are the benchmarks without AVX2 support:
```rust
test naive            ... bench:      42,295 ns/iter (+/- 1,435)
test rotate32         ... bench:      19,726 ns/iter (+/- 2,989)
```
and here are the results with AVX2 enabled:
```rust
test naive            ... bench:      45,362 ns/iter (+/- 1,511)
test rotate32         ... bench:       7,635 ns/iter (+/- 150)
test pext256_avx      ... bench:      25,339 ns/iter (+/- 1,671)
```

Disappointing, to say the least! It turns out the Rust compiler is much better at
unrolling the loop with SIMD extensions than we are, so with AVX2 enabled our
base implementation is going to be parallelized much better than what we could
do. The only time we can do better is when BMI2 is available and not AVX...

...but that never happens since the AVX2 instruction set is older than the BMI2
one, so any recent CPU has both of these. *\*Sigh.\**

# Conclusion

I tried some other implementations, some iterating in reversed order (hoping to
replace an `inc/cmp/jne` block with a `dec/jnz` block), some shifting
inplace: turns out the algorithm described above is one of the fastest ! Below
are the benchmark with the `avx` and `bmi2` CPU features enabled, run on an `i7-8550U` CPU:

```rust
running 10 tests
test naive            ... bench:      59,947 ns/iter (+/- 10,536)
test naive_rev        ... bench:      55,028 ns/iter (+/- 8,745)
test pext256_avx      ... bench:       9,712 ns/iter (+/- 1,895)
test pext32_bmi2      ... bench:      24,336 ns/iter (+/- 3,568)
test pext64_bmi2      ... bench:      17,815 ns/iter (+/- 3,204)
test rotate32         ... bench:       9,800 ns/iter (+/- 1,459)
test rotate32_asm     ... bench:      58,054 ns/iter (+/- 6,336)
test rotate32_inplace ... bench:       9,948 ns/iter (+/- 1,497)
test rotate32_rev     ... bench:      11,111 ns/iter (+/- 1,452)
test rotate64         ... bench:      12,881 ns/iter (+/- 1,916)
```

With `rotate32` being that close to `pext256_avx`, I assume the Rust compiler is
actually unrolling our loop and using SIMD in the backend, while our code is
also much more simple than an equivalent one using inline assembly. **That's
several hours spent trying to optimise something, and failing to beat the Rust
compiler. *\*Yay!\****
