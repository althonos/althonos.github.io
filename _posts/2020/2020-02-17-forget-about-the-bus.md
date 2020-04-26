---
layout: post
title:	"Forget about the bus, worry about the knife."
date:	2020-02-17 17:22:00
categories:
    - blog
    - dev
tags:
    - coding
    - me
    - misc
---

*This is not an article about [D-Bus](https://en.wikipedia.org/wiki/D-Bus), sorry.*

The **bus factor** has been theorized in the context of software development as
an easy question to grasp one of the main issues of specialization in coding
teams: **how much would your project be impacted if a given team member was to
be [hit by a bus](https://legacy.python.org/search/hypermail/python-1994q2/1040.html) tomorrow?**

<figure class="fullwidth">
  <iframe src="https://www.youtube.com/embed/r35mGCF8db8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen class="fullwidth" height="100%"></iframe>
  <figcaption class="figure-caption">
    And if a double-decker bus crashes into us...    
  </figcaption>
</figure>

Now, this is an actual problem, since it is extremely easy as a developer to
stop communicating about your code unless you have peer-pressure that makes you
do so. Writing code a certain way becomes an habit, and developers are already
quite opinionated sometimes about the need for comments in their code, or with
the adoption of a particular technology. However, the usual solution to this
problem you'll find online is always the same: *just cross-train everyone and
reduce the complexity of your code*. Fine.

Or actually, not fine. The issue of the **bus factor** is that it is often an
[*post-hoc* understanding of a failure](https://en.wikipedia.org/wiki/Post_hoc_analysis)
during the early stages of development. Starting to develop a project only to
realize later that some people became key in its development is, in my
unexperienced opinion, quite obvious. Even more, this shows that project
organisation has failed to reduce team partitioning beforehand. Building a
narrative on the individual responsibility when it is more of a collective
result is not the most brilliant way to solve the issue.

Now for my personal experience: the idea of bus factor is nonexistent in
scientific programming. Most of the software is built by independent people
that write it alone, publish it, and then move on. Sure,
[this does not result in the best quality](https://academia.stackexchange.com/questions/17781/why-do-many-talented-scientists-write-horrible-software),
but hey, academia has its own standards. Scientific programmers cannot get
away from the bus factor problem because, as any other scientists, most of
them end up reaching a specialization level only five or ten people in the
entire world also have. The academic system won't give you funding to simply
train someone to replace you; they'll expect that someone to come up with
something new, and often something you will not fully understand
even as their supervisor.

The result of this is what I would call the **knife factor**: *what's the point
of writing things? What's the point of going to work to write software that
only a handful of people will ever use?* I sometimes wished to get something
more out of a project than a paper; I don't care about papers actually, that's
probably going to hurt my scientific career. But the result is the same;
academia is not going to ask what the **bus factor** of your project is, because
[there is a high chance it's **one**](https://zenodo.org/record/495360)... or,
more often: they do not even know your project.

[**If nobody worries about you being hit by a bus, who worries about you cutting
your wrists open?**](https://www.registerednursing.org/suicide-rates-profession/)
