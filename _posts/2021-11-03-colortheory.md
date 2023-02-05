---
layout: post
title:  "Color Theory - Green Screen & Chroma Keying"
date:   2021-11-03
author: Christian Robles
category: blog
published: true
---

In my Multimedia Systems Engineering course we have an assignment to implement a chroma key algorithm. Our problem was to implement a chroma keying program that was flexible enough to handle a wide variety of colors with gradient backgrounds, without per-input tuning.

I'm not thrilled about the amount of time I spent chasing bad solutions - including a large chunk of time trying to implement ["High Quality Chroma Key"](https://web.archive.org/web/20180930132543/http://www.cs.utah.edu/~michael/chroma/) and others. There are many chroma key solutions that produce high quality results in YCrCb space but require hand-tuning; the constraints of this project eliminate these options as good candidates.

Instead, we use a hueristic approach. We convert the RGB images into HSV space, and filter pixels out of the background that land within certain H, S, and V bands based off of specific parameters. I won't detail the precise hueristics in case someone has found this and is trying to spoil the assignment :^)

Here are my results -

<img src="/assets/images/chromakey/bufforig.png" alt="shia" width="250"/>
<img src="/assets/images/chromakey/buffkeyed.png" alt="shia" width="250"/>

<img src="/assets/images/chromakey/gusorig.png" alt="gus" width="250"/>
<img src="/assets/images/chromakey/guskeyed.png" alt="gus" width="250"/>

<img src="/assets/images/chromakey/ninjaorig.png" alt="ninja" width="250"/>
<img src="/assets/images/chromakey/ninjakeyed.png" alt="ninja" width="250"/>

There's some trouble at the edges, but for a flexible hueristic approach I'm happy with these results. I also implemented a Gaussian blur filter to smooth out jaggedness a bit.
