---
layout: post
title:	"Mapping the PICO-8 memory"
date:	2019-06-18 18:50:00
categories:
    - blog
    - pico8
tags:
    - coding
    - rust
    - pico8
---

The PICO-8 API is super tightly coupled to its *memory layout*: that is,
functions are just more abstract ways of interacting with the memory directly.
For instance, writing on the screen can be done with the `pset` function,
or directly by writing in the `0x6000`..`0x7FFF` range of the memory.

After checking the wiki as well as the
[code](https://www.lexaloffle.com/bbs/?tid=34561)
[of](https://www.lexaloffle.com/bbs/?tid=33414)
[some](https://www.lexaloffle.com/bbs/?tid=31976)
[cartridges](https://www.lexaloffle.com/bbs/?tid=29529)
on the [Lexaloffle BBS](https://www.lexaloffle.com/bbs/), I realised it was
necessary for an accurate implementation to have the same exact layout as the
reference one.

A very thorough description of the memory can be found in the [Memory page of
the PICO-8 wiki](https://pico-8.fandom.com/wiki/Memory).

The `picoxyde-mem` crate is the first take at an implementation of the PICO-8
memory structure, using `#[repr(C)]` types that are only composed of `u8` fields
of array. I'm also using [`plain`](https://docs.rs/plain/) to directly access
some parts of the memory in a controlled way. The getters and setters for most
fields is derived with the help of [`getset`](https://docs.rs/getset).

```rust
/// The PICO-8 RAM.
#[repr(C)]
#[derive(Setters, MutGetters, Getters, Clone)]
pub struct Ram {
    #[get = "pub"]
    #[get_mut = "pub"]
    #[set = "pub"]
    rom: Rom,
    /* ... */
    #[get = "pub"]
    #[get_mut = "pub"]
    #[set = "pub"]
    screen: ScreenSegment,
}
```

The PICO-8 is [little-endian](https://en.wikipedia.org/wiki/Endianness), so we
cannot store `i16` numbers directly (since we want the memory storage to be
platform agnostic), but the logic can be abstracted away with the
[`byteorder`](https://docs.rs/byteorder) crate: for instance, this is
how the `0x5f28..0x5f2b` memory range (draw camera position) is declared:

```rust
#[repr(C)]
#[derive(Clone)]
pub struct RamCameraPosition {
    x: [u8; 2],
    y: [u8; 2],
}

impl RamCameraPosition {
    pub fn set(&mut self, x: i16, y: i16) {
        LittleEndian::write_i16(&mut self.x, x);
        LittleEndian::write_i16(&mut self.y, y);
    }
}
```

The whole RAM can be addressed directly using the `peek` and `poke` functions,
but since all types are `u8` or `[u8; N]` we are never violating possible
assumptions made by Rust, and our types are valid at all time: in other words,
the memory implementation is **safe** !
