---
layout: post
title:  "Generating OSL Shaders with MaterialX"
date:   2022-03-16
author: Christian Robles
category: blog
tags: graphics
published: true 
---

Last weekend I kicked off my spring break with a visit to Sequoia National Park. We saw some beautiful big trees and stopped at Bishop Peak on our return trip to get kicked by some stiff postwar boulders.

<img src="/assets/images/mtlx-to-osl/bigtree.png" alt="big trees" width="250"/>
<img src="/assets/images/mtlx-to-osl/bigrock.png" alt="big boulders" width="250"/>

Having shaken off some spring semester burnout, I spent the past few days figuring out how to generate OSL shaders from *.mtlx files using MaterialX. In this post I'll give an overview of MaterialX, share the snippets I used to generate OSL shader source, and show some examples of using the generated shaders in Blender.

## MaterialX

> [MaterialX](https://github.com/AcademySoftwareFoundation/MaterialX) is an open standard for representing rich material and look-development content in computer graphics, enabling its platform-independent description and exchange across applications and renderers. Launched at Industrial Light & Magic in 2012, MaterialX has been a key technology in their feature films and real-time experiences since Star Wars: The Force Awakens and Millennium Falcon: Smugglers Run.

MaterialX is an open standard for representing surface materials. The repository holds some [sample .mtlx files](https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/resources/Materials/Examples/StandardSurface), which can be viewed using the [MaterialX Viewer](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/documents/DeveloperGuide/Viewer.md). The viewer seems to have some components that don't play well with the M1 chip, so I built it with x86 rosetta.

<img src="/assets/images/mtlx-to-osl/mtlx-viewer.png" alt="viewer" width="250"/>
<img src="/assets/images/mtlx-to-osl/mtlx-viewer-opts.png" alt="viewer opts" width="250"/>

Using the viewer, we can show some sample materials. For this example, I'll take the [`standard_surface_gold.mtlx`](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/resources/Materials/Examples/StandardSurface/standard_surface_gold.mtlx) and translate it to OSL. Here's how the gold material looks on [Dennis](https://free3d.com/3d-model/dennis-posed-004-812878.html) -

<img src="/assets/images/mtlx-to-osl/dennis-gold.png" alt="dennis gold" width="505"/>

## Generating OSL

To generate OSL source code from a .mtlx file, we can use [MaterialX Shader Generation](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/documents/DeveloperGuide/ShaderGeneration.md). To my understanding, a *.mtlx document holds a nodegraph describing the properties of some material. The Shader Generator uses a language specific [generator context](https://github.com/AcademySoftwareFoundation/MaterialX/blob/main/source/MaterialXGenShader/GenContext.cpp) to translate a document nodegraph to shader source code. The resulting source code can then be written to a file to be consumed by other renderers.

I put together a small console wrapper for OSL shader generation. The source is available here - [https://github.com/roblesch/mtlx-to-osl](https://github.com/roblesch/mtlx-to-osl). This is just a small demonstrative app, so it has some smelly pieces like requiring absolute input paths and no optional configurations for shader generation. I may expand this down the line if there is any value in doing so.

The code was put together from snippets in the MaterialX Viewer. It's more or less a pared down [`loadDocument()`](https://github.com/AcademySoftwareFoundation/MaterialX/blob/b58042eae7e2c866f0b9b36c2476f0f846ee0df8/source/MaterialXView/Viewer.cpp#L1153) - most of the difficulty came from figuring out which configuration I didn't need. 

You can find a sample generated OSL file in `mtlx-to-osl/samples/standard_surface_gold.osl`. Generated OSL shaders depend on `mx_funcs.h` being present in the same directory as the shader.

## Previewing OSL Shaders in Blender

With [Blender](https://www.blender.org/), we can preview our generated OSL code. We can load our source as a [script node](https://docs.blender.org/manual/en/latest/render/shader_nodes/osl.html) and attach its output to a mesh. Here's a scene I put together with the generated OSL on the mesh of Dennis, in a diffuse box with some emissive surface lights. Rendered with Cycles, 1024 samples, denoised.

<img src="/assets/images/mtlx-to-osl/osl-script-node.png" alt="script node" width="505"/>

<img src="/assets/images/mtlx-to-osl/dennis-gold-render.png" alt="viewer opts" width="505"/>

The blender scene is available in `samples/blender-sample.blend`. You may need to re-connect the OSL script nodes.
