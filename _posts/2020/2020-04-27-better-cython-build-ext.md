---
layout: post
title:	"Better setuptools integration for Cython project"
date:	2020-02-17 17:22:00
categories:
    - blog
    - dev
tags:
    - coding
    - cython
    - python
---

I recently wrote my first [Cython](https://cython.org/) extension, and
it's really great. I would strongly discourage someone without C experience
to use it, because it's basically letting you do all the stupid mistakes C programmers know about; but when you know how to use
[`memchr`](http://man7.org/linux/man-pages/man3/memchr.3.html),
[`strstr`](http://man7.org/linux/man-pages/man3/strstr.3.html), and the likes,
you really feel like you're expanding on Python.


## Compiling without optimizations in debug mode

When you're developing a Cython extension, you end up compiling it a lot.
Unfortunately, `setuptools` builds extensions with the
[`-O3`](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html) level at all
times, and that can greatly increase the build time for a large codebase, or
a project using very large files (this can happen in scientific programming,
where it is common to use very large parameter tables written as C headers).

To solve this issue, we can override the `build_ext` command so that it sets
the optimization flag based on the `--debug` flag. Add the following class
declaration to `setup.py`:
```python
# setup.py
from setuptools.command.build_ext import build_ext as _build_ext

class build_ext(_build_ext):

    def build_extension(self, ext):
        if self.debug:
            ext.extra_compile_args.append("-O0")
        _build_ext.build_extension(self, ext)
```

And then, register that new command when calling `setup`:
```python
# setup.py
setuptools.setup(
    ext_modules=cythonize(...),
    cmdclass={"build_ext": build_ext},
)
```

Depending on the `--debug` flag, the extension will be built with opt-level 0
or 3. Note that you can also compile in debug mode by default by putting the
debug flag as true directly inside `setup.cfg`:

```toml
# setup.cfg

[build_ext]
debug = true
```


## Enabling coverage

Of course, as a serious programmer, you're writing unit tests and you want to
measure the coverage of your code. To do so, you need to tweak some
configuration files so that it works.

First, you need to enable the `Cython.Coverage` plugin so that coverage can be
collected from your compiled code. For this, add the following lines to the
`setup.cfg` file at the root of your project:

```toml
# setup.cfg

[coverage:run]
plugins = Cython.Coverage
```

Then, you need to have Cython generate a line trace. Add the following directive
to the top of all of your `.pyx` files:
```python
# cython: linetrace=True
```

Finally, we need to have `setuptools` compile with linetrace support, by defining
the `CYTHON_TRACE` macro when compiling. Actually, unless the code becomes
unrealistically slow, it's even better to define `CYTHON_TRACE_NOGIL` to even
have coverage of code running outside the GIL.

However, `CYTHON_TRACE` should only be declared when building in debug mode,
since it adds some overhead to the generated code. To do so, the easiest is to
override the `build_ext` command, which is used to build the Cython extension.
If we go back and edit our custom `build_ext` class, we get:
```python
# setup.py
import sys
from setuptools.command.build_ext import build_ext as _build_ext

class build_ext(_build_ext):

    def build_extension(self, ext):
        if self.debug:
            ext.extra_compile_args.append("-O0")
            if sys.implementation.name == "cpython":
                ext.define_macros.append(("CYTHON_TRACE_NOGIL", 1))
        _build_ext.build_extension(self, ext)
```

Note that we check for the platform before enabling the Cython trace, as line
trace support is unavailable for other platforms such as PyPy.

Congratulations! You can now measure the coverage on your Cython extension
module with the two following lines (this uses `unittest` by you could use
any test runner, really):
```console
$ python setup.py build_ext --debug --inplace
$ python -m coverage run -m unittest discover -vv
```
