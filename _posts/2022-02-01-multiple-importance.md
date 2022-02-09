---
layout: post
title:  "Shortest Path to Multiple Importance Sampling"
date:   2022-02-05
author: Christian Robles
tags: graphics
published: true 
---

Since mid December I have been reading Eric Veach's 1997 Ph.D. Thesis [*Robust Monte Carlo Methods for Light Transport Simulation*](http://graphics.stanford.edu/papers/veach_thesis/). I've recently been working on extending Peter Shirley's implementation of a Monte Carlo path tracer from his book [*Ray Tracing: The Rest Of Your Life*](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html) with the Multiple Importance Sampling techniques proposed by Veach as a side project. After about 7 weeks of staring at formulas and banging my head into things, I think I've got something to share (a whole week ahead of schedule too!)

In this post I'll share the route I took through Veach's thesis and how I implemented Multiple Importance Sampling with a Balance Heuristic in Peter Shirley's path tracer. Shortest path is a bit of a stretch. The source for this project is [available here](https://github.com/roblesch/multiple-importance-sampling).

Many personal thanks to Mike Day for recommending the readings that gave this project its legs, Peter Shirley for entertaining too many questions, and to Professor Neumann for some generous encouragement.

- [Overview](#overview)
- [Light Transport](#light-transport)
- [Monte Carlo Methods](#monte-carlo-methods)
  - [Advantages of Monte Carlo Methods](#advantages-of-monte-carlo-methods)
- [Importance Sampling](#importance-sampling)
- [Multiple Importance Sampling](#multiple-importance-sampling)
- [Implementation](#implementation)
  - [Implementing MIS](#implementing-mis)
  - [Notes](#notes)
- [Sources](#sources)

<br>

## Overview

Eric Veach's 1997 Ph.D. Thesis [*Robust Monte Carlo Methods for Light Transport Simulation*](http://graphics.stanford.edu/papers/veach_thesis/) presents novel integration techniques that address the highly complex nature of light transport algorithms. Multiple Importance Sampling is one such technique that improves the robustness of Monte Carlo algorithms by using more than one sampling method to evaluate an integral and creating a weighted combination of the results.

I'll share some discussion on Light Transport, Monte Carlo methods and Importance Sampling before talking through Multiple Importance Sampling. Then I'll cover the changes I've made to Peter Shirley's code and show some images I've created. My discussion of these topics will generally be conversational, but each section will have links to readings if you are interested in theory.

The primary sources for this post are [Eric Veach's thesis](http://graphics.stanford.edu/papers/veach_thesis/) and Peter Shirley's [Ray Tracing Series](https://raytracing.github.io/), with some excerpts from [Physically Based Ray Tracing](https://www.pbrt.org/).

<br>

## Light Transport

<a href="https://www.pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/The_Light_Transport_Equation"><img src="https://www.pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/path-annotated-1.svg" alt="pbrt" width="505"/></a>

> The light transport equation (LTE) is the governing equation that describes the equilibrium distribution of radiance in a scene. It gives the total reflected radiance at a point on a surface in terms of emission from the surface, its BSDF, and the distribution of incident illumination arriving at the point. [PBRT 14.4](https://www.pbr-book.org/3ed-2018/Light_Transport_I_Surface_Reflection/The_Light_Transport_Equation)

In computer graphics, photorealistic rendering techniques rely on approximations of the behaviors of light to produce physically convincing depictions of 3d scenes. Given a description of the geometry and material properties of a scene, the problem is to simulate the behavior of light as it travels through the scene, bouncing between material surfaces on its path from light sources before eventually being evaluated at the camera and producing the resulting image. Light transport describes the mathematical approximations for the various physical behaviors that can occur when light interacts with some interacting media.

The Light Transport Equation is the unifying equation that describes the exitant radiance at a point on a surface in terms of its incident radiance and the properties of the surface. Light Transport can be formulated as an integral over transport paths (Veach 8, 8.2), allowing us to use Monte Carlo methods to evaluate the values at each pixel in an image.

More information:
- Veach 3, 4, 8
- PBRT 14

<br>

## Monte Carlo Methods

My first exposure to the Monte Carlo method was from Thomas J. Sargent's online lecture series [*Quantitative Economics with Julia*](https://julia.quantecon.org/getting_started_julia/julia_by_example.html#exercise-3) where as an exercise, he asks the reader to solve the common problem of computing $$\pi$$ using Monte Carlo. You can find a great summary of this exercise in [Shirley 2](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#asimplemontecarloprogram)

Here is nice a video that illustrates this idea -

<iframe width="560" height="315" src="https://www.youtube.com/embed/I_plXHHKPCo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<br>

### Advantages of Monte Carlo Methods

This process for estimating $$\pi$$ is an example of [Monte Carlo Integration](https://en.wikipedia.org/wiki/Monte_Carlo_integration). Monte Carlo methods are helpful when integrals are difficult to calculate directly. We can use Monte Carlo methods to approximate the value of integrals with high dimensions or whose underlying functions are not known precisely. Monte Carlo methods are useful in solving the Light Transport problem since light transport can be formulated as an integral over transport paths (Veach 8), and the high dimensionality is poorly suited to numerical methods. 

Other mathematical advantages to Monte Carlo methods include convergence in $$O(N^{-1/2})$$ (*Veach 2.4.1*), generalization to domains that are not well-suited to analytical solutions (*Veach 2.4*) and application to integrands with singularities (*Veach 2.4*).

More information:
- Shirley 2, 3, 4
- Veach 2
- PBRT 13

<br>

## Importance Sampling

With Monte Carlo methods, the quality of our approximation scales with the number of samples we evaluate. Importance sampling allows us to converge to a good approximation by focusing sampling on the regions that provide the most information. In a nutshell, if we want to evaluate the integral of some function $$f$$, it may be the case that the probability of sampling $$f$$ in a region that provides little information is very high. Importance sampling allows us to focus our evaluation on the regions of $$f$$ which provide the most information by changing the distribution that we sample from, reducing the samples needed to converge on a good approximation.

In light transport our analogous sampling of regions that provide more information are more literal. Remember that our goal is to simulate the path of light throughout a scene, so ideally every path we sample should terminate at a light. Unfortunately, by purely random surface scattering this will rarely be the case. However, if a surface is in direct lighting, that is that its path to a light is unobstructed, then that light will likely provide the most information on the appearance of the surface, so we should trace paths toward the light. If we can't see a light, we could scatter according to the properties of the surface, since we may find a path to the light eventually. This is exactly what is meant when I reference "sampling the lights" or "sampling the BRDF" in later sections.

Here is a very good video on the subject of importance sampling. It's made for a Machine Learning audience, but the theory is the same -

<iframe width="560" height="315" src="https://www.youtube.com/embed/C3p2wI4RAi8" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

More information:
- Shirley 2, 3, 6
- Veach 2, 3, 4, 9
- PBRT 13.10

<br>

## Multiple Importance Sampling

Take a look at these three images, each generated with 16 samples per pixel -

<img src="/assets/images/mis/brdf-16.png" alt="brdf-16" width="200"/>
<img src="/assets/images/mis/lights-16.png" alt="lights-16" width="200"/>
<img src="/assets/images/mis/mis_brdf-16.png" alt="mis-brdf-16" width="200"/>

The image on the left was generated by sampling the BRDF only. We can see that sampling the BRDF is very noisy at few samples. It converges on an approximation of the color of a pixel very slowly, since the chance of finding our way to the light by random scattering is very low.

The image in the middle was generated by sampling the lights only. It converges on an approximation of the color of a pixel very quickly, since we will always sample a path directly to the light. However, we can see that in locations where no path to the light exists - in the shadow of the boxes or on the ceiling - we are not able to gather any information at all.

The image on the right was generated using multiple importance sampling. In this technique we take the best of both worlds - in cases where a path to the light exists, we sample the light. Otherwise, we scatter according to the properties of the material. With multiple importance sampling, we use a weighted combination of multiple sampling techniques to produce a result. There are many ways to calculate weights (Veach 9.2), but here we use the balance heuristic -

$$
w_s(x) = \frac{p_s(x)}{\sum_i p_i(x)}
$$

Where $$p_s(x)$$ is the evaluation of some pdf, and $$\sum_ip_i(x)$$ is the some of the evaluation of all pdfs used. In this case, $$p_s$$ is either the pdf of the lights or the BRDF, and $$\sum_ip_i(x)$$ is the sum of these two values. The color is then approximated by multiplying the resulting $$w_s$$ by the value of the color calculated by sampling each path and combining the results.

More information -
- Veach 9
- PBRT 13.10.1

<br>

## Implementation

I began with [the source from](https://github.com/RayTracing/raytracing.github.io/tree/master/src/TheRestOfYourLife) Peter Shirley's *Ray Tracing: The Rest of Your Life*. In Peter's implementation, he provides a [Mixture PDF](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#mixturedensities/themixturepdfclass) that samples paths using an equal 50/50 mix of the pdfs of the [BRDF](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#lightscattering/thescatteringpdf) and the [lights](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#samplinglightsdirectly) with [russian roulette](https://www.pbr-book.org/3ed-2018/Monte_Carlo_Integration/Russian_Roulette_and_Splitting) path termination.

Mixture pdf, 16, 64, 256 samples per pixel

<img src="/assets/images/mis/mixed-16.png" alt="mixed-16" width="200"/>
<img src="/assets/images/mis/mixed-64.png" alt="mixed-64" width="200"/>
<img src="/assets/images/mis/mixed-256.png" alt="mixed-256" width="200"/>


To make implementation of MIS a little easier, I made some small simplifications:

- Removed support for [motion blur](https://raytracing.github.io/books/RayTracingTheNextWeek.html#motionblur) and [textures](https://raytracing.github.io/books/RayTracingTheNextWeek.html#solidtextures)
- Colors are written to `image.ppm`
- Simplify the Cornell Box scene to two lambertian boxes

<br>

### Implementing MIS

To demonstrate Multiple Importance Sampling, I modify the `ray_color` function to create the following alternate functions.

`Le(ray, scene, lights)`

This function separates the evaluation of the [BRDF](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#lightscattering/thescatteringpdf) from `ray_color`, and samples paths using only samples from the BRDF.

Sampling BRDF, 16, 64, 256 samples per pixel

<img src="/assets/images/mis/brdf-16.png" alt="brdf-16" width="200"/>
<img src="/assets/images/mis/brdf-64.png" alt="brdf-64" width="200"/>
<img src="/assets/images/mis/brdf-256.png" alt="brdf-256" width="200"/>

`Ld(ray, scene, lights)`

This function separates the evaluation of the [lights](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html#samplinglightsdirectly) from `ray_color`, and samples paths using only samples from the lights.

Sampling Lights, 16, 64, 256 samples per pixel

<img src="/assets/images/mis/lights-16.png" alt="lights-16" width="200"/>
<img src="/assets/images/mis/lights-64.png" alt="lights-64" width="200"/>
<img src="/assets/images/mis/lights-256.png" alt="lights-256" width="200"/>


`pe(ray, scene)`

This function evaluates the pdf of the BRDF at a given point.

`pd(ray, scene)`

This function evaluates the pdf of the lights at a given point.

`main()`

Multiple Importance Sampling occurs in the main loop as follows -

```c
for each pixel
    color pixel_color(0, 0, 0);
    for each sample // N samples
        ray = camera.get_ray();
        pd_i = pd(ray, scene);
        pe_i = pe(ray, scene);
        w_d = BalanceHeuristic(pd_i, pe_i);
        w_e = BalanceHeuristic(pe_i, pd_i);
        pixel_color += w_d * Ld(ray, scene, lights);
        pixel_color += w_e * Le(ray, scene, lights);
    endfor
    pixel_color /= N;
endfor
```

Using this implementation yields the following results -

Multiple Importance, Lights + BRDF, Balance Heuristic, 16, 64, 256 samples per pixel

<img src="/assets/images/mis/mis_brdf-16.png" alt="mis-brdf-16" width="200"/>
<img src="/assets/images/mis/mis_brdf-64.png" alt="mis-brdf-64" width="200"/>
<img src="/assets/images/mis/mis_brdf-256.png" alt="mis-brdf-256" width="200"/>

Multiple Importance, Lights + Mixture PDF, Balance Heuristic, 16, 64, 256 samples per pixel

<img src="/assets/images/mis/mis_mixed-16.png" alt="mis-mixed-16" width="200"/>
<img src="/assets/images/mis/mis_mixed-64.png" alt="mis-mixed-64" width="200"/>
<img src="/assets/images/mis/mis_mixed-256.png" alt="mis-mixed-256" width="200"/>

### Notes

We can see that Multiple Importance Sampling allows us to utilize different sampling techniques where they are most effective. We gain the benefit of converging to a solution more quickly in areas where the light is visible by sampling the lights directly, and cover areas where the lights cannot be sampled by sampling the BRDF or the Mixture PDF. This technique could be further improved by using something like [adaptive sampling](https://web.cs.wpi.edu/~matt/courses/cs563/talks/antialiasing/adaptive.html) to determine the main contributor in a pixel area, and alter the samples per pixel to sample more densely in areas where the light is not visible, and sample more sparsely in areas where it is.

## Sources

[Peter Shirley, _Ray Tracing: The Rest of Your Life_](https://raytracing.github.io/books/RayTracingTheRestOfYourLife.html)

[Matt Pharr, Wenzel Jakob, Greg Humphreys, _Physically Based Rendering_](https://www.pbrt.org/)

[Eric Veach, _Robust Monte Carlo Methods for Light Transport Simulation_](http://graphics.stanford.edu/papers/veach_thesis/)