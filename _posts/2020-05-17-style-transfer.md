---
layout: post
title:  "Recreating Effects from Aphex Twin's \"T69 Collapse\""
date:   2020-05-17
author: Christian Robles
category: blog
published: true
---

<iframe width="505" height="280" src="https://www.youtube.com/embed/SqayDnQ2wmw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Recently I was watching the Music video for Aphex Twin's "T69 Collapse" and I was curious about some effects -

<figure>
<a href="/assets/images/style-transfer/building-2.png"><img src="/assets/images/style-transfer/building-2.png" alt="building" width="40%"/></a>
<a href="/assets/images/style-transfer/building.png"><img src="/assets/images/style-transfer/building.png" alt="building" width="40%"/></a>
<figcaption>Screenshots from 0:34, 1:45</figcaption>
</figure>

The video was created by [Weirdcore](http://weirdcore.tv/) - you can find his post about it [here](http://weirdcore.tv/2018/08/07/aphex-twin-collapse-video/). It includes links to interviews where Weirdcore discusses his process, use of photogrammetry in his works, and how he achieved some of the effects in T69 Collapse through the use of Style Transfer.

## Photogrammetry

I dug up an [interview with Weirdcore](https://thequietus.com/articles/25150-weirdcore-interview-radiohead-aphex-twin-caretaker), the artist behind the music video.

>...I just did loads of photogrammetry of all different places and put it all together. It looks like one street, but it’s loads of different locations. Photogrammetry is where you basically take loads of pictures of something from different angles and feed it into this software that recreates it in 3D. It’s loads of 3D models built from photographs.

What is photogrammetry? I dug up Wilfried Linder's book [Digital Photogrammetry](https://link.springer.com/content/pdf/10.1007/978-3-540-92725-9.pdf) -

>Photogrammetry provides methods to give you [quantitative data from photo measurements] ... photogrammetry can be defined as the "science of measuring in photos" ... the basic task is to get object co-ordinates of any point in the photo from which you can then calculate geometric data or create maps.

>  If we have two (or more) photos from the same object but taken from different positions, we may easily calculate the three-dimensional co-ordinates of any point which is represented in both photos. Therefore we can define the main task of photogrammetry in the following way: For any object point represented in at least two photos we have to calculate the three-dimensional object (terrain) co-ordinates.

## Style transfer

In his [interview with Fast Company](https://www.fastcompany.com/90216189/how-aphex-twins-t69-collapse-video-used-a-neural-network-for-hallucinatory-visuals) Weirdcore discusses his use of [Transfusion.AI](https://transfusion.ai/) to warp the textures of the photogrammetry collage. From their about page, Transfusion.AI is a style transfer framework bundled as a package for After Effects.

What is style transfer? [Image Style Transfer Using Convolutional Neural Networks](https://openaccess.thecvf.com/content_cvpr_2016/papers/Gatys_Image_Style_Transfer_CVPR_2016_paper.pdf) by Gatys and others says -

> Transferring the style from one image onto another can be considered a problem of texture transfer. In texture transfer the goal is to synthesise a texture from a source image while constraining the texture synthesis in order to preserve the semantic content of a target image.

> To obtain a representation of the *style* of an input image, we use a feature space designed to capture texture information ... we obtain a representation of the input image which captures its texture information but not the global arrangement. 

> To transfer the style of an artwork $$\vec{a}$$ onto a photograph $$\vec{p}$$ we synthesise a new image that simultaneously matches the content representation of $$\vec{p}$$ and the style representation of $$\vec{a}$$.

## Recreating the effect

I thought it would be interesting to see if I could apply style transfer to a 3d scan without doing any programming. 

To capture a 3d scan, I found the app [Scandy Pro 3D Scanner](https://apps.apple.com/us/app/scandy-pro-3d-scanner/id1388028223) on the app store. It worked alright, the fidelity is about what you could expect. The app gives a model and some textures. I loaded the model into [Blender](https://www.blender.org/) and attached the textures for some interesting results -


<img src="/assets/images/style-transfer/face-raw.png" alt="face" width="40%"/>
<img src="/assets/images/style-transfer/face.png" alt="face" width="40%"/>

For style transfer, I found [Arbitrary Style Transfer in the Browser](https://reiinakano.com/arbitrary-image-stylization-tfjs/). I ran the scandy textures through style transfer on a couple of their sample images -

<img src="/assets/images/style-transfer/clouds-src.jpeg" alt="face" width="40%"/>
<img src="/assets/images/style-transfer/face-clouds.png" alt="face" width="40%"/>

<img src="/assets/images/style-transfer/sketch-src.jpeg" alt="face" width="40%"/>
<img src="/assets/images/style-transfer/face-sketch.png" alt="face" width="40%"/>

Out of curiosity I ran style transfer with the original texture as the style image -

<img src="/assets/images/style-transfer/face-texture.png" alt="face" width="40%"/>
<img src="/assets/images/style-transfer/face-recurs.png" alt="face" width="40%"/>

Here's what the transferred textures look like mapped back onto the models -

<video src="/assets/images/style-transfer/faces-rotate.mp4" controls="controls" width="80%">
</video>

Not exactly as compelling as the original, but pretty neat!