---
layout: post
title:  "Summer of Standards"
date:   2022-09-02
author: Christian Robles
published: true
---

<img src="/assets/images/summer22/banner.png" alt="banner"/>

This summer I had the opportunity to work with Autodesk's Graphics Platform Team on an extension for the Academy Software Foundation's [MaterialX](https://materialx.org/). Our goal was to support translation from Autodesk's [Standard Surface](https://autodesk.github.io/standard-surface/) material model to the Khronos Group's [glTF 2.0](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html) Physically Based Rendering (PBR) [Metallic-Roughness](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#materials) model. The project was done in Open Source, and was an exciting opportunity to engage with some of the most significant contributors in the open material standards community.

This post is a longer form of some slides I delivered as my internship wrap-up. You can see the original deck [here](/assets/images/summer22/roblesc_intern_pres.pdf).

#### Links

- [Slide deck](/assets/images/summer22/roblesc_intern_pres.pdf) - internship wrap-up slides

#### Organizations

- [Autodesk](https://www.autodesk.com/) - Authors of Standard Surface and my employer for the summer ♡.
- [Academy Software Foundation (ASWF)](https://www.aswf.io/) - governing body for a [family of Open Source projects](https://landscape.aswf.io/).
- [Khronos Group](https://www.khronos.org/) - Open consortium collaborating on open standards for 3D graphics.

#### Standards

- [glTF 2.0](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html) - Khronos Group's standard for 3D content delivery. "The JPEG of 3D".
- [MaterialX](https://materialx.org/) - Open Standard governed by ASWF for material authoring and transmission.
- [Standard Surface](https://autodesk.github.io/standard-surface/) - Autodesk's material model specification.

#### Libraries

- [MaterialX](https://github.com/AcademySoftwareFoundation/MaterialX) - Utilities for `*.mtlx` I/O, sample Viewer with test assets
- [dspbr-pt](https://github.com/DassaultSystemes-Technology/dspbr-pt) - Dassault Systèmes Enterprise PBR Sample Renderer
- [Arnold (Kick)](https://docs.arnoldrenderer.com/display/A5AFMUG/Kick) - Ray tracing renderer for feature animation and VFX
- [cgltf](https://github.com/jkuhlmann/cgltf) - Single-file C glTF loader and writer

## Project Overview

In MaterialX, materials are represented as a graph of nodes, where each node may be some closure of a physically based behavior (for example, a [dielectric bsdf](https://github.com/AcademySoftwareFoundation/MaterialX/blob/4f44b5a0e465ba6ec9abe8f246b504517fa32efa/libraries/pbrlib/pbrlib_defs.mtlx#L58)), an arithmetic operator node, or utilities for channel operations, etc. When material authoring is complete, the nodegraph can be processed by [MaterialX Shader Generation](https://materialx.org/docs/api/md_documents__developer_guide__shader_generation.html), to output a shader (OSL, glsl, hlsl, etc) that can be consumed by a renderer. I have a blog post on [generating OSL shaders with MaterialX](/2022/03/16/mtlx-to-osl.html).

The MaterialX standard can encode many material models, and exposes single-node interfaces for some models which abstract away the underlying nodegraphs. Standard Surface and glTF PBR are two such models. This work aims to map Standard Surface material model inputs to the glTF PBR model resulting in a material that adheres to the glTF PBR model but maintains visual parity with its Standard Surface native equivalent. Additionally, MaterialX encoding of a glTF material can be exported to the [native glTF 2.0 JSON](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#schema-reference-material) to provide an end-to-end asset export pipeline from Standard Surface material producers to native glTF consumers.

## Translation & Reference Renders

<img src="/assets/images/summer22/translation.png" alt="translation"/>

Material Translation occurs in three stages starting from a MaterialX encoding of a Standard Surface material. This material's inputs are mapped to a translation nodegraph, which maps them to a glTF PBR node. This stack of Standard Surface:translation node:glTF PBR nodes are processed by [MaterialX texture baking](https://github.com/AcademySoftwareFoundation/MaterialX/blob/50d45c146c959a6891dc4140a7eae792f094e2d1/source/MaterialXRenderGlsl/TextureBaker.cpp). The flattened glTF PBR node is then exported to native glTF with cgltf.

This project focused on the implementation of the translation nodegraph and enhancements to native glTF export. The nodegraph achieved approximate parity for core features, and further enhancements for greater visual parity between Standard Surface and glTF PBR are still ongoing.

<img src="/assets/images/summer22/rendering.png" alt="rendering"/>

Each stage of materials is consumed by a renderer for reference. Standard Surface MaterialX files are consumed by Arnold directly, and glTF PBR materials are consumed by Arnold after shader generation to OSL. Native glTF materials are consumed by dspbr-pt.


<img src="/assets/images/summer22/batching.png" alt="rendering"/>

Orchestrating translation and rendering is straightforward - MaterialX offers Python bindings for its utilities via [pybind11](https://github.com/pybind/pybind11). A set of test materials gathered from the [AMD GPUOpen MaterialX library](https://matlib.gpuopen.com/main/materials/all) are enumerated and processed, generating the intermediate materials on each step from MaterialX Standard Surface to native glTF. Arnold renders are automated with the Kick API, and dspbr-pt via its command line.

## Final Thoughts

This work spanned a wide set of challenges, and was a very deep crash course in Open Material Standards. It was a great experience collaborating with contributors across the community - I'm keeping in touch with ASWF as I work on further enhancements to the translation nodegraph.

Last but not least, I owe a huge thanks to my manager Sankar Ganesh and mentor Nicolas Savva. Thank you for being engaged and for always having some new resources to share. And of course, thank you for giving me the opportunity to work with your team for the summer. :-)
