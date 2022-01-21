---
layout: post
title:  "Shortest Path to Multiple Importance Sampling"
date:   2022-01-01
author: Christian Robles
tags: graphics
published: false
---

Eric Veach's 1997 Ph.D. Thesis [*Robust Monte Carlo Methods for Light Transport Simulation*](http://graphics.stanford.edu/papers/veach_thesis/) presents novel integration techniques that address the highly complex nature of light transport algorithms. Multiple Importance Sampling is one such technique that improves the robustness of Monte Carlo algorithms by using more than one sampling method to evaluate an integral and creating a weighted combination of the results.

In this post I'll discuss the critical path that forms my understanding of Multiple Importance Sampling. Following the structure of Veach's work, I'll first give some short background on the Light Transport problem which will help motivate the need for Monte Carlo methods and contextualize the advantages of multiple importance sampling. I'll then relate my understanding of Monte Carlo methods and Light transport with respect to Veach's thesis, supplemented with excerpts from other materials. Last I'll share a small path tracer I've put together to demonstrate the effects of Multiple Importance Sampling.

In addition to Veach's thesis and various Wikipedia pages, other notable sources include [Physically Based Rendering](https://www.pbr-book.org/3ed-2018/contents) and Peter Shirley's [Ray Tracing in One Weekend](https://raytracing.github.io/), which were especially leaned on for the assembly of the code sample.

## Light Transport

> Light transport theory deals with the mathematics behind calculating the energy transfers between media that affect visibility. [Wikipedia](https://en.wikipedia.org/wiki/Light_transport_theory)

In computer graphics, photorealistic rendering techniques rely on approximations of the behaviors of light to produce physically convincing depictions of 3d scenes. Given a description of the geometry and material properties of a scene, the problem is to simulate the behavior of light as it travels through the scene, bouncing between material surfaces on its path from light sources before eventually being evaluated at the viewport and producing the resulting image. Light transport broadly encompasses the mathematical approximations for the different physical behaviors that can occur when light interacts with some surface.

Even for relatively simple scenes with few objects, we can see that approximating the behavior of light is a difficult task. Light can bounce off of surfaces it encounters (reflection) or pass through them (transmission), but it can also exhibit more complex behaviors like surfaces passing their color off to nearby objects ([diffuse reflection](https://en.wikipedia.org/wiki/Diffuse_reflection)) or surfaces that scatter the path of light internally ([subsurface scattering](https://en.wikipedia.org/wiki/Subsurface_scattering)). Not only this, but each of these behaviors encompass a spectrum where they can be exhibited weakly or strongly. 

It is clear to see that attempting to simulate all these behaviors is a complex task, even moreso as the number of objects in a scene increases. Due to this complexity, it is critical to design algorithms which can arrive at close approximations efficiently and with low error.

The focus of this post is a technique where we focus the evaluation of approximations on the spaces that provide the most information, called Multiple Importance Sampling.

## Monte Carlo Methods

### Approximating $$\pi$$

My first exposure to the Monte Carlo method was from Thomas J. Sargent's online lecture series [*Quantitative Economics with Julia*](https://julia.quantecon.org/getting_started_julia/julia_by_example.html#exercise-3) where as an exercise, he asks the reader to compute $$\pi$$ using Monte Carlo. 

To summarize the exercise, first consider two random values $$x$$ and $$y$$ on $$[-1,1]$$. These values describe a point $$p(x,y)$$ and that $$p$$ lies some distance $$d$$ from the origin. The possible $$p$$ lie within a $$2\times 2$$ square with area $$4$$. 

Next consider a circle inscribed within the square centered on the origin $$0,0$$ with radius $$r = 1$$ and area $$\pi r^2 = \pi$$. We can say that a point $$p$$ lies in the circle if its distance from the origin is less than the radius of the circle $$d \leq r$$.

We know the radius of the circle is $$1$$, we know area of is $$\pi$$, and we know the area of the enclosing square is $$4$$, so we can say that the probability that a random point $$p$$ lies in the circle is $$\pi/4$$.

Knowing this, we can then estimate $$\pi$$ by dropping many random points on $$[-1,1]$$ and checking if they land in the circle - the ratio of the points that land in the circle to the total number of points will approximate $$\pi/4$$, and the accuracy of our approximation will improve as the number of points increases. 

Here is nice a video that illustrates this idea -

<iframe width="505" height="280" src="https://www.youtube.com/embed/I_plXHHKPCo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Advantages of Monte Carlo Methods

This process for estimating $$\pi$$ is an example of [Monte Carlo Integration](https://en.wikipedia.org/wiki/Monte_Carlo_integration). Monte Carlo methods are helpful when measures are difficult to calculate directly. For example, exact calculations of $$\pi$$ require [evaluating an infinite series](https://www.mathscareers.org.uk/calculating-pi/). In comparison, our Monte Carlo approach was relatively simple. We needed only to find the average of some $$n$$ samples of picking a random point and computing its distance from the origin.

Other mathematical advantages to Monte Carlo methods include convergence in $$O(N^{-1/2})$$ (*Veach 2.4.1*), generalization to domains that are not well-suited to analytical solutions (*Veach 2.4*) and application to integrands with singularities (*Veach 2.4*).

<!-- ### Monte Carlo integration

In *Veach 2.4*, Veach establishes basic Monte Carlo integration as follows -

> 

> The idea of Monte Carlo integration is to evaluate the integral

$$
I = \int_\Omega f(x) d\sigma(x)
$$

> using random sampling. In its basic form, this is done by independently sampling $$N$$ points $$X_1, ..., X_N$$ according to some convenient density function $$p$$, and then computing the estimate

$$
F_N = \frac{1}{N}\sum_{i=1}^{N}\frac{f(X_i)}{p(X_i)}
$$

> Here we have used the notation $$F_N$$ rather than $$\hat{I}$$ to emphasize that the result is a random variable, and that its properties depend on how many sample points were chosen. -->
