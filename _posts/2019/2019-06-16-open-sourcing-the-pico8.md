---
layout: post
title:	"Open-sourcing the PICO-8: a foreword"
date:	2019-06-16 10:16:00
categories:
    - blog
    - pico8
tags:
    - coding
    - pico8
---

It all started with **love**.

Or, to be more accurate, with the [LOVE 2D engine](https://love2d.org/), that
caught my attention at the end of the year 2018. At that time, I was finishing
the third semester of my Master's Degree in Bioinformatics, had a lot of time,
and few ideas. As a classic case of
[**RIIR** (Rewrite It In Rust)](https://transitiontech.ca/random/RIIR),
I decided to try my hand at implementing the engine in Rust on top of another
game engine (with Piston in mind). I did the implementation of a small part of
the API, wrote my first
[proc-macros](https://blog.rust-lang.org/2018/12/21/Procedural-Macros-in-Rust-2018.html),
and ultimately lost interest.

The [PICO-8](https://www.lexaloffle.com/pico-8.php) pushes things forward.
Where LOVE is a game engine, the PICO-8 is a fantasy console, and has much more
constrained specifications. All of this is is *virtual*, but makes sense when
considering the actual consoles of the 8-bit era.

As a proud owner of a (hacked) **Nintendo Switch**, I thought it would be great to
have it being able to play PICO-8 games: I was having great moments playing older
games through emulation on that device. Thing is, PICO-8 is closed-source
(even though the games are open-source when distributed in cartridge format),
and I hardly see [@zep](https://twitter.com/lexaloffle) starting to support the
Switch homebrew scene...

I started looking for emulators or alternate implementations. A quick look on
GitHub made me stumble on the following projects:

* `UnicornConsole`: started as an open-source PICO-8 implementation, it became
  its own thing after that.
* `LIKO-12`: built on top of love, not a PICO-* implementation.
* `picolove`: a partial implementation of the PICO-8 API directly in LOVE. It is
  however **not memory compatible** with the actual PICO-8, and I was going to
  learn that it is a huge problem.

Since none of these was a good starting point for my usecase, I decided to write
my own implementation. I am relying on the following resources:

* The [official PICO-8 manual](https://www.lexaloffle.com/pico8_manual.txt),
  which describes the public API of the console, as well as some parts of the
  memory
* The [PICO-8 wiki](https://pico-8.fandom.com/wiki/Pico-8_Wikia) that has better
  description of some undocumented behaviours in the reference implementation.
* [`picotool`](https://github.com/dansanderson/picotool/), a Python toolbox that
  can decode PICO-8 cartridges and extract Lua code, spritesheets, and more.

Note that I ***do not own*** the PICO-8, so I can't use it to check the correctness
of my implementation directly. I am instead using cartridges from the BBS that
are publicly released to check my code.
