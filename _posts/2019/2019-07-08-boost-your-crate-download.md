---
layout: post
title:	"Boost your crates.io download stats with this one simple trick!"
date:	2019-07-08 16:19:00
categories:
    - blog
    - devops
tags:
    - travis-ci
    - rust
---

Some people get happy by the number of likes they get on their Instagram
picutres. If you're reading this article, you're most likely not one of them;
but I assume you still get a small endorphin boost whenever someone stars your
repository on GitHub.

Here is just a simple trick to increase your download count on [`crates.io`](https://crates.io):
when setting up Travis-CI, **do not use `cargo` caching**! Instead, use the
`directories` option to simply cache the following folder:
```yaml
# .travis.yml

cache:
  directories:
    - $HOME/.cargo/bin
    - $TRAVIS_BUILD_DIR/target
```

Doing so will ensure any binary you install in the CI environment is cached
(this can be useful for instance if you're using
[`cargo-make`](https://sagiegurari.github.io/cargo-make/)), and that the
build artefacts from last run are kept as well.

**However**, by *not* caching the whole `$HOME/.cargo` folder, we prevent Travis-CI
from keeping the `crates.io` registry and the source of our dependencies between
each run, and they are downloaded again at every run! This has no cost on the
compilation time since the sources are only *downloaded*, not *recompiled*: once
all sources have been fetched, they are matched against the incremental build
folder that has been cached, and unless a new version is available the built
libraries are used as-is.

Doing so in a project that depends on one of your crates will effectively
increase your download stats (provided you are doing active development on
the dependent project)!

<figure class="fullwidth">
  <img src="{{ 'images/2019/2019-07-08/downloads.png' | relative_url }}" class="fullwidth"/>
  <figcaption class="figure-caption">
    A thousand downloads for a crate I'm the only one actually using? Hell yeah!
    *please send help I shouldn't be excited about that*
  </figcaption>
</figure>
