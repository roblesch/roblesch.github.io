---
layout: post
title:  "Revisiting Ray Tracing"
date:   2021-11-28
author: Christian Robles
published: true
---

In my Computer Graphics course our end of term project was the implementation of a simple ray tracer. After working [Dr. Ramamoorthi's course](/2020/10/25/edx-graphics.html) and [Peter Shirley's In One Weekend Series](/2020/05/26/raytracing-weekend.html), implementing a ray tracer was much less painful, though I am still vulnerable to math-typos.

For this assignment we required ray-sphere and ray-triangle intersection, phong shading, phong lighting, and shadow rays. For extra credit I've also implemented soft shadows and anti aliasing by averaging the result of casting some number of rays randomly in a radius around each pixel and from each intersection towards a radius around each point light source.

Here's some outputs -

<img src="/assets/images/rtrevisited/table.png" alt="table" width="505"/>

<img src="/assets/images/rtrevisited/spheres.png" alt="spheres" width="505"/>

With soft shadows and anti aliasing disabled -

<img src="/assets/images/rtrevisited/siggraph.png" alt="siggraph" width="505"/>
