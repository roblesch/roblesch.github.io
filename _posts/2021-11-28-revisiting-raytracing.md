---
layout: post
title:  "Revisiting Ray Tracing"
date:   2021-11-28
author: Christian Robles
published: true
---

In my Computer Graphics course our end of term project was the implementation of a simple ray tracer. After working [Dr. Ramamoorthi's course](/2020/10/25/edx-graphics.html) and [Peter Shirley's In One Weekend Series](/2020/05/26/raytracing-weekend.html), implementing a ray tracer was much less painful, though I am still vulnerable to math-typos. (It took me way too long to realize I was forgetting to shoot the ray from my point of intersection when calculating shadow rays.)

For this assignment we required ray-sphere and ray-triangle intersection, phong shading, phong lighting, and shadow rays. For extra credit I've also implemented soft shadows and anti aliasing; for both of these approaches I've just cast some extra random rays per pixel and averaged the result.

Here's some of my outputs -

<img src="/assets/images/rtrevisited/table.png" alt="table" width="505"/>

<img src="/assets/images/rtrevisited/spheres.png" alt="spheres" width="505"/>

With soft shadows and anti aliasing disabled -

<img src="/assets/images/rtrevisited/siggraph.png" alt="siggraph" width="505"/>
