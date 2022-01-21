---
layout: post
title:  "Deadmau5 and PyTorch - Evaluating Real-Time Style Transfer in TouchDesigner"
date:   2020-06-16
author: Christian Robles
published: true
---

I've been a long time [deadmau5](https://deadmau5.com) fan. Last January I had the chance to see him perform at the House of Blues in Boston as part of his [cube v3 tour](https://cubev3.com/). There's a great behind the scenes video where he talks about about the software that powers the show -

<iframe width="505" height="280" src="https://www.youtube.com/embed/H2fDbkXoVZs" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

The software is called [TouchDesigner](https://derivative.ca/) and is developed by Derivative. In May I played with [Style Transfer and 3d Scanning](/2020/05/17/style-transfer.html), and I was curious if it would be possible to implement a custom TouchDesigner operator to perform style transfer on video inputs in real time.

## TouchDesigner

There's a great [introduction series by Ben Voigt](https://www.youtube.com/watch?v=wmM1lCWtn6o) that walks through TouchDesigner fundamentals. In short, TouchDesigner is a software built for processing signals, generally audio or video. TouchDesigner projects (or networks) consist of operators (nodes) that output data to other operators. Signals are transformed with operators such as [TOP](https://docs.derivative.ca/TOP), [CHOP](https://docs.derivative.ca/CHOP), or [SOP](https://docs.derivative.ca/SOP), and then sent to outputs, like the screen of your laptop or the surface of deadmau5's cube.

<img src="/assets/images/touchdesigner/touchdesignerproject.png" alt="touchdesigner" width="505"/>

TouchDesigner supports [Custom Operators](https://docs.derivative.ca/Custom_Operators) and has a [repository of examples](https://github.com/TouchDesigner/CustomOperatorSamples). Before jumping into writing a custom operator, I thought it would be best to check out other TouchDesigner projects and see if real time style transfer is a viable goal.

## Real Time Style Transfer

A quick search yields exsstas's repository ["StyleTransfer-in-TD"](https://github.com/exsstas/StyleTransfer-in-TD), and [this post on the community forums](https://derivative.ca/community-post/fast-style-transfer-c-top). Both seem to have issues with framerate; exsstas uses [this repository](https://github.com/lengstrom/fast-style-transfer/) for style transfer that mentions videos can [be composed from merging stills](https://github.com/lengstrom/fast-style-transfer/#video-stylization). 

To make life easy I tried zhangzhang1989's [PyTorch-Multi-Style-Transfer](https://github.com/zhanghang1989/PyTorch-Multi-Style-Transfer) which includes an out of the box real-time demo using a webcam. To get the demo running I needed the dependencies `opencv-python`, `torch`, `torchvision` and `torchfile`.

Unfortunately, even running on my GPU (GTX 1060) I'm only able to manage ~5 - 10 frames/sec. On CPU, it takes several seconds to produce a single frame. Unfortunately, this project will stay on the shelf for the time being - maybe some future advances in style transfer research will bring it back to life. Regardless, TouchDesigner is a great piece of software and I hope to have more excuses to poke around.
