---
layout: post
title:	"Adding fixed-point arithmetic to the Lua interpreter"
date:	2019-06-18 18:50:00
categories:
    - blog
    - pico8
tags:
    - coding
    - rust
    - C
    - pico8
---

Consoles from the 8-bit era didn't have float-processing unit, so the
operation on real numbers had to be made using the integer registers, using
**[fixed-point arithmetic](https://en.wikipedia.org/wiki/Fixed-point_arithmetic)**.
PICO-8 reproduces that behaviour using Q16.16 numbers: 16 bits are used to
store the integer part, and 16 bits are used for the fractional part.

    0x      XXXX    .    XXXX
            |--|         |--|
       integer part   fractional part

For instance, the maximum unsigned number you can store is `0xFFFF.FFFF`:

$$
\begin{array}{llll}
    FFFF.FFFF_{16} & = & FFFF_{16} &+& \frac{FFFF_{16}}{2^{16}} \\
                   & = & 65535  &+& \frac{65535}{65536} \\
                   & = & 65535&.&99998474121
\end{array}
$$

This is basically equivalent to store a number `n` in memory as the closest
$$n \times 2^{16}$$ integer: the additions and subtractions are trivial, but
other operations require some tweaking.

## Fixed-point in C

Although I could just have started from scratch using `i32_t` to store Q16.16
numbers, the [`libfixmath`](https://github.com/PetteriAimonen/libfixmath) library
already provides the common operations on Q16.16 already. `libfixmath` provides
a `fix16_t` type that aliases `i32_t`.

Fortunately, Lua has support for different float implementations, and most of
these can be changed using the `LUA_FLOAT_TYPE` define, so we should be able
to use `fix16_t` with minimal changes.
The [`luaconf.h`](https://www.lua.org/source/5.3/luaconf.h.html) header file
defines macros that provide the required number interface. For instance,
here is how Lua can be compiled with `long long` support for reals:

```cpp
#if LUA_FLOAT_TYPE == LUA_FLOAT_LONGDOUBLE

  /* LUA_NUMBER is the floating-point type used by Lua. */
  #define LUA_NUMBER      long double

  /* l_mathlim(x) corrects limit name 'x' to the proper float type
   * by prefixing it with one of FLT/DBL/LDBL.
   */
  #define l_mathlim(n)            (LDBL_##n)

  /* LUAI_UACNUMBER is the result of a 'default argument promotion'
   * over a floating number.
   */
  #define LUAI_UACNUMBER  long double

  /* @@ LUA_NUMBER_FRMLEN is the length modifier for writing floats.*/
  #define LUA_NUMBER_FRMLEN       "L"

  /* LUA_NUMBER_FMT is the format for writing floats. */
  #define LUA_NUMBER_FMT          "%.19Lg"

  /* l_mathop allows the addition of an 'l' or 'f' to all math operations.
  #define l_mathop(op)            op##l

  /* lua_str2number converts a decimal numeric string to a number. \*/
  #define lua_str2number(s,p)     strtold((s), (p))

#endif
```

We simply have to declare `LUA_NUMBER` and `LUAI_UACNUMBER` to be `fix16_t`,
and change the `l_mathop` declaration to use all `fix16_` operations from
the `libfixmath`. There's is however a problem here: we can't use
`LUA_NUMBER_FRMLEN` since we are not using a builtin type anymore.

I added a `l_litint` macro to build a number from a literal, and changed the
definition of the other types as well to use `l_litint` instead. Same thing
for `LUA_NUMBER_FMT`: we can't use `sprintf` anymore. I `undef`'ed these
two macros to find all their uses in the rest of the C code, and patched it
there directly.

The `libfixmath` functions don't behave exactly like the one from the
POSIX C library, so I had to fix that in the `libfixmath` source as well:

* I added a new `strtofix16` to behave more or less like
  [`strtod`](https://linux.die.net/man/3/strtod), accepting numbers in
  hexadecimal representation and returning the current buffer position.
* I patched `fix16_to_str` to return the size of the generated string like
  [the `printf` family of functions](https://linux.die.net/man/3/sprintf).

The mathematical operations are then declared in the `llimits.h` header:
```c
#define luai_numadd(L,a,b)      ((a)+(b))
#define luai_numsub(L,a,b)      ((a)-(b))
#define luai_nummul(L,a,b)      ((a)*(b))
#define luai_numdiv(L,a,b)      ((a)/(b))
```
and we replace them with the `fix16` variants (under an `ifdef` branch):
```c
#define luai_numadd(L,a,b)      (fix16_add(a, b))
#define luai_numsub(L,a,b)      (fix16_sub(a, b))
#define luai_nummul(L,a,b)      (fix16_mul(a, b))
#define luai_numdiv(L,a,b)      (fix16_div(a, b))
```

## Adding the fixed-point Lua to `rlua`

[`rlua`](https://docs.rs/rlua) is a great library that provides a safe Rust
abstraction over the Lua interpreter.
