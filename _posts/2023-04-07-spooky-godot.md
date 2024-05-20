---
layout: post
title:  "A Spooky Weekend in Godot"
date:   2023-04-07
author: Christian Robles
category: blog
image: "/assets/images/walking-sim/skull-bob.mp4"
published: true
---

<figure>
<video autoplay muted loop playsinline>
  <source src="/assets/images/walking-sim/skull-bob.mp4" />
  Your browser does not support the video tag.
</video>
<figcaption>Where'd he come from? Where's he going?</figcaption>
</figure>
If you've been seeing my spam on LinkedIn, you'll know I've been supremely busy pushing out work on my [Directed Research](/blog/2022/11/17/directed-research.html) project. At this point I'm a week or two away from wrapping it all up, so I think the best thing I could do now is start a new project (ha, ha).

I've been working in Unity for awhile now on the development of [Grandma Green](/blog/2023/05/11/grandmagreen.html). There's a lot to enjoy about working in Unity - it's very easy to prototype and iterate on new ideas, and the perforce integrations have worked well enough for our team.

All this time spent in Unity has put other engines on my radar. Most of my recent work is research focused where a lot of time invested into questionably designed, purpose-built programs that demonstrate some specific technique but have little application anywhere else. Game Engine architectures are a compelling topic - most 3d engines will have to orchestrate concurrent execution of rendering, animation, input handling, game entities & behaviors, networking, etc.

### Surveying Game Engines

There are... [a lot of game engines...](https://en.wikipedia.org/wiki/List_of_game_engines) but a few with recent releases stood out.

<div class="gallery-grid">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/d/da/Unreal_Engine_Logo.svg/150px-Unreal_Engine_Logo.svg.png" class="no-shadow"/>
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/O3DE_Color_Logo.svg/220px-O3DE_Color_Logo.svg.png" class="no-shadow"/>
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/5/5a/Godot_logo.svg/200px-Godot_logo.svg.png" class="no-shadow"/>
</div>
<br>

Odds are you don't need me to tell you about Unreal Engine. Unreal 5 has some very real [GDC '23 hype](https://www.unrealengine.com/en-US/events/gdc2023), so I thought I'd give it a shot. A staggering 3 hours of shader compilation on install tested my patience, and waiting another 3 hours to have all those shaders re-compile when I created a new project was enough to earn it a spot on the shelf for now. It's an incredible platform, but doesn't feel like a good fit for weekend experimentation.

Open 3D Engine [(o3de)](https://github.com/o3de/o3de) is an open-source 3D game engine developed by the Open 3D Foundation, a subsidiary of the Linux Foundation. O3de began life as Amazon Games's in-house engine Lumberyard, which was itself based on CryEngine. Besides being backed by Amazon games, o3de boasts a long list of industry partners including Adobe, Huawei, Red Hat and SideFX. O3de has an [upcoming 23.05 release](https://docs.o3de.org/docs/release-notes/2305-0-release-notes/) that boasts a spread of improvements, but my read on the community take is that it still has a way to go befure it's mature enough for general use. I haven't actually given it a shake, so take that with a grain of salt.



[Godot](https://github.com/godotengine/godot) is a FOSS, MIT-licensed game engine. The recent 4.0 release caught my attention for its addition of support for the Vulkan API and introducing Signed Distance Field Global Illumination [(SDFGI)](https://docs.godotengine.org/en/stable/tutorials/3d/global_illumination/using_sdfgi.html) to its graphics pipeline. At a staggering download size of ~50MB and requiring no installation, it's hard not to give Godot a try.

### Godot in 30 Seconds

Projects in Godot are organized into scenes of nodes. Every scene is a tree of nodes with one root node. A scene can be included in another hierarchy as a node (conceptually similar to a prefab in Unity). A node can be anything - an actor, a light, a static mesh, various geometries, bundles of data, etc. Godot can be used to build 2D or 3D games, and comes with a lot of useful 3D primitives and behaviors (various meshes, collision, lighting) out of the box. Custom behaviors are written in GDScript, Godot's custom scripting language that behaves a lot like Python.

[> Godot Docs](https://docs.godotengine.org/en/stable/getting_started/step_by_step/nodes_and_scenes.html)

## What are we making?

Why is the hardest part always picking something to make?

I took some inspiration from the recent [Dungeon Crawler Jam 2023](https://itch.io/jam/dcjam2023). I was especially impressed by Ankehotep's Prison, so I set out to make a simple desert-themed low-poly dungeon with some kind of actors that follow the player around.

Behold, a convenient tutorial -

<div class="iframe-wrapper">
  <iframe allowfullscreen class="responsive-iframe" src="https://www.youtube.com/embed/VjuyfBaryu8"></iframe>
</div>

### Environment & Models

I got some desert-y textures from [textures.com](https://www.textures.com/search?q=desert) and followed along with the tutorial to put together a tileset to use in Godot. I had some trouble getting the export settings right to get Godot to accept the tileset from Blender due to some version mismatching - downgrading to Blender 3.5 fixed the export.

<img src="/assets/images/walking-sim/tileset.jpg" width="80%">

Painting a tilemap was very easy. Throw on some lighting, turn on the volumetric fog. Why not throw in a random Angel statue from sketchfab? That's starting to look kinda spooky.

<img src="/assets/images/walking-sim/dungeon.jpg" width="80%">

It's pretty incredible how fast you can turn an idea into a playable prototype with an engine like Godot. I try not to think about it when I'm fighting to get simple features working in my own engine...

I've had this idea kicking in the back of my mind of some kind of oscillating skeleton/floating golem, now seems like a good excuse to play with it. I found a low-poly skull model on sketchfab where the jaw and skull are separated. I keyframed a simple hinging animation with Blender -

<video muted controls playsinline width="80%">
  <source src="/assets/images/walking-sim/skull-blender.mp4" />
  Your browser does not support the video tag.
</video>

Using the asset in Godot was simple enough - from Blender I can export as gltf with the keyframed animation. Importing it to Godot works out of the box.

### Simple behaviors

Some simple behaviors will be enough to wrap things up for the weekend. Godot's physics system takes care of simple collisions, just add some transparent collider meshes to the player, skull and statue. I was surprised at how well the collisions with the gltf tilemap worked without any refinement, though there was some noise on the floor - adding a slightly raised ground plane was simple enough.

The player controller is a simplified snippet from the First Person Controller on the Godot asset marketplace. It walks around with WASD, looks around with the mouse. I stripped out other behaviors and added a simple crouch by shifting the camera & collapsing the collision mesh when the user presses l-ctrl.

For the skull I added some simple player tracking behavior to move towards the player until it is some distance away, then orbit around. Throw in some extra sine waves to make it bob up and down and modulate the direction it faces for some extra character. And a purple light, for style.

The Angel statue gets a simple [weeping angel](https://tardis.fandom.com/wiki/Weeping_Angel) behavior that moves towards the player while the player is not looking at them. Visibility checks are solved easily through the aptly named 

```
$Visibility.is_on_screen()
```

Movement and orientation towards the player are achieved by manipulating the statue's basis matrix in the direction of the vector between the statue's position and the position of the player.

Put it all together, and it's a nice little dungeon... thingy...

<figure>
<video muted controls loop playsinline width="80%">
  <source src="/assets/images/walking-sim/spooky-gameplay.mp4" />
  Your browser does not support the video tag.
</video>
<figcaption>Neat!</figcaption>
</figure>

It was really easy to get this going with Godot. No major hiccups; moving assets between Blender and Godot with gltf was especially easy. Past Python experience made picking up GDScript a breeze, and the engine exposes a lot of properties for common tasks (movement, orientation, on-screen checks) that are straightforward to manipulate.

Unfortunately that's all the time I have for this project, it's back to the Light Transport mines. I would definitely pick up Godot for future prototypes or Jams, it was a lot of fun to work with for the weekend!