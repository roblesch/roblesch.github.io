---
layout: post
title:  "Shortest Path to Multiple Importance Sampling"
date:   2022-01-01
author: Christian Robles
tags: graphics
published: false
---

Eric Veach's 1997 Ph.D. Thesis [*Robust Monte Carlo Methods for Light Transport Simulation*](http://graphics.stanford.edu/papers/veach_thesis/) presents novel integration techniques that address the highly complex nature of light transport algorithms. Multiple Importance Sampling is one such technique that improves the robustness of Monte Carlo algorithms by using more than one sampling method to evaluate an integral and creating a weighted combination of the results.

In this post I'll trim the fat off my last couple month's reading and discuss the critical path that forms my understanding of Multiple Importance Sampling. I'll start by summarizing Veach's presentation of Monte Carlo methods and Light transport supplemented with some excerpts from other materials, then I'll share a small path tracer I've put together to demonstrate the effects of Multiple Importance Sampling.

Throughout this post I'll be supplementing my discussion of Veach's content with excerpts from [*Physically Based Rendering*](https://www.pbr-book.org/) and the [*Ray Tracing In One Weekend*](https://raytracing.github.io/) series, as well as Wikipedia and other sources. The code sample especially relies heavily on source code adapted from these sources.

Since this is a shortest path, much of the concrete theory will be condensed or omitted. If you want to convince yourself of the theory (or give this post a much needed gut check), I'll do my best to link to the relevant chapters of Veach's work as I discuss them.

## Monte Carlo Methods

My first exposure to the Monte Carlo method was from Thomas J. Sargent's [*Quantitative Economics with Julia*](https://julia.quantecon.org/getting_started_julia/julia_by_example.html#exercise-3) where as an exercise, he asks the reader to compute $$\pi$$ using Monte Carlo. 

To summarize the exercise, consider a circle centered on the origin $$0,0$$ with radius $$r = 1$$ and area $$\pi r^2 = \pi$$.

Consider two random values $$x$$ and $$y$$ on $$[-1,1]$$. The point $$p(x,y)$$ is some distance $$d$$ from the origin, and the possible values of $$p$$ lie within a square with area $$(1 - (-1))^2 = 4$$. 

We can say that $$p$$ lies in the circle if the distance from the origin is less than the radius, or $$d \leq 1$$. We know the area of the circle is $$\pi$$, and we know the area of the enclosing square is $$4$$, so we can say that the probability of a random point landing in the circle is $$\pi/4$$.

Knowing this, we can then estimate $$\pi$$ by dropping many random points on $$[-1,1]$$ and checking if they land in the circle - the ratio of the points that land in the circle to the total number of points will approximate $$\pi/4$$, and the accuracy of our approximation will improve as the number of points increases. 

Here is nice a video that illustrates this idea -

<iframe width="560" height="315" src="https://www.youtube.com/embed/I_plXHHKPCo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

