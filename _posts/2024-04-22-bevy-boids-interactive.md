---
layout: post
title:  "Interactive Boids Simulation"
date:   2024-04-22
author: Christian Robles
category: blog
image: ""
published: true
---
<figure>
    <canvas id="bevy_boids_canvas"></canvas>
    <figcaption>
        Use your mouse to guide the boids. Performance degrades when app loses focus (try clicking it).<br>
        <a href="https://github.com/bevyengine/bevy/issues/4078">Multithreading is unavailable in the browser.</a>
    </figcaption>
    <script type="module">
        import init from '/assets/bevy_spatial_boids/bevy_spatial_boids.js';
        init().catch((error) => {});
    </script>
</figure>

Made for the browser in Rust with [Bevy WASM](https://bevy-cheatbook.github.io/platforms/wasm.html).

Blog post: [Rust K-D Tree Boids on Bevy with bevy-spatial](/blog/2024/04/29/bevy-boids.html).

Repository: [roblesch/bevy_spatial_boids](https://github.com/roblesch/bevy_spatial_boids)