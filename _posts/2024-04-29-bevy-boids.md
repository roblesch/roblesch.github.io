---
layout: post
title:  "Rust K-D Tree Boids on Bevy with bevy-spatial"
date:   2024-04-29
author: Christian Robles
category: blog
image: "/assets/images/boids/boids-preview-wide.jpg"
published: true
---

<figure>
    <a href="/blog/2024/04/22/bevy-boids-interactive.html"><img src="/assets/images/boids/boids-preview-wide.jpg" /></a>
    <figcaption>Boids! <a href="/blog/2024/04/22/bevy-boids-interactive.html">Try them in your browser.</a></figcaption>
</figure>

- [Reading & Rambling](#reading--rambling)
  - [What Doth Life](#what-doth-life)
  - [Back To The Building Blocks](#back-to-the-building-blocks)
  - [The Cathedral and the Bazaar](#the-cathedral-and-the-bazaar)
  - [Bevy](#bevy)
- [Boids](#boids)
  - [Entities, Components, Systems](#entities-components-systems)
  - [Boid Lookup Acceleration](#boid-lookup-acceleration)
  - [The Devil in the Details](#the-devil-in-the-details)
  - [Multithreading](#multithreading)
  - [Building for the Web](#building-for-the-web)
  
## Reading & Rambling

The first section of this post is a ramble about project selection, getting started with Rust and Bevy, and Eric S. Raymond's *The Cathedral and the Bazaar*. If you want the technical details about implementing boids on Bevy, skip [here](#boids).

### What Doth Life

After wrapping the first iteration of my [Vulkan renderer](/blog/2023/08/05/palace-1.html) I had the opportunity to interview for a Graphics Programmer position. I spent a few weeks integrating [Nvidia's Variable-Rate Shading algorithm](https://github.com/NVIDIAGameWorks/nas-sample) into a test scene as a take-home project. The interview was a success, but they were unable to accommodate me remotely, and I was unable to move. These things happen. Here's [the slides](https://blog.roblesch.page/assets/VRS-NAS.pdf) for posterity.

I spent a long while thinking about my Next Big Project:TM:. I started with refactoring the renderer in an effort to move it towards some simple game engine functionality. The renderer is tightly coupled to the Vulkan API; it's not much more than a mish-mash of tutorials with a glTF parser and an interactive camera. I started designing some middleware to abstract Vulkan and, eventually, other APIs like DirectX or Metal. Combing through the source for Unreal Engine and Godot gave great direction, but I had an overwhelming feeling that I was reinventing the wheel.

I was interested in experimenting with WebGPU, and I really wanted to explore the ECS paradigm. Still, I wasn't sure about committing to building a full engine from scratch - it's great to build something for the enjoyment of learning, but was I certain about committing thousands of hours of development over many years to build (yet another) esoteric hobby engine just for the sake of building one? I decided to shelve these plans for the time being and do something new to find some direction.

### Back To The Building Blocks

It was around this time the White House Office of the National Cyber Director published [*Back to the Building Blocks: A Path Toward Secure and Measurable Software*](https://www.whitehouse.gov/wp-content/uploads/2024/02/Final-ONCD-Technical-Report.pdf). The report describes an urgent need to address undiscov- wait a second, did the White House just publish a report recommending *Rust*? The Rustaceans danced in the streets. The C++ programmers wept. The time had finally come.

Though not quite the dawn of a new era, appearing in a White House report was strong support, so it seemed as good a time as any to jump on the Rust train. I was going in circles on other projects, and learning about Rust's famous memory safety could inform how I program in other languages. Learning never fails. I visited the [Rust](https://www.rust-lang.org/) site and found their book, [*The Rust Programming Language*](https://doc.rust-lang.org/book/) - otherwise ominously referred to as *"The Book"* (and if you look a little deeper, there's [*"The Rustonomicon"*](https://doc.rust-lang.org/nomicon/) where all the juicy secrets are hiding).

The Book is a concise overview of the major features of Rust, paired with bite-sized code snippets to demonstrate. I downloaded JetBrains's [RustRover Preview](https://www.jetbrains.com/rust/) which feels very familiar after the time spent with CLion. After going through the Book and writing this boids program (I promise I'll get there eventually), my first impressions are very positive. It's easy to get started with Rust, and it has some great ideas to bring to the table.

Good
- The toolchain. `rustup` and `cargo` make project setup incredibly easy.
- The Book. Rust's library of learning materials is excellent and highly accessible.
- Ergonomic standard library. Feels like good pieces from many modern languages.

Meh
- Cumbersome function signature syntax. Did [lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime/explicit.html) need to look like that?
- Efficiency often necessitates violating memory safety with `unsafe` code.

Overall, I'm really enjoying writing Rust, and I can see why it's [so consistently loved](https://survey.stackoverflow.co/2023/#section-admired-and-desired-programming-scripting-and-markup-languages). There is a wealth of resources, the community is welcoming, and the tooling is excellent.

### The Cathedral and the Bazaar

I'll be honest, I can't retrace my path to *The Cathedral and the Bazaar*. I was scrolling through Apache's open source community guidelines in bed late one night and somehow ended up there, but now I can't find it mentioned on a single Apache community page.

Some way or another, I ended up reading Eric S. Raymond's 1997 essay on the Linux approach to open source, *The Cathedral and the Bazaar*. The essay contrasts two software development models - the *Cathedral*, where a small community of developers makes source available on each release, but development is otherwise opaque (Emacs, GCC) - and the *Bazaar*, where all development happens publicly (Linux). These comparisons are supported with the author's personal experience applying the Bazaar development model to his own project, Fetchmail. It's a mix of hacker philosophy and practical suggestions on cultivating community around an open source project. The veracity of the practical propositions [are disputed](https://news.ycombinator.com/item?id=35939383), but I think the essay's popularity indicates that something resonated with a lot of people at the time, so it's an interesting peek into the movements that led to today's software landscape. 

The core ideas are around following your nose, surrounding yourself with fresh eyes that will provide new stimuli, and pushing out code with the assurance that you can't get it right the first try. These spoke directly to my recent project rut - I was performing all my development in hermitage, emerging every few months to share a blog post or some screenshots on Discord for some scraps of feedback. I've been stuck in design paralysis for quite some time, going in circles weighing tradeoffs for largely aesthetic desicions. I think it's inevitable that I'll come back to building projects in this space, but for now it's time to see what's out there in open source.

### Bevy

A couple weeks of learning Rust and sifting through 90's wisdom had me feeling invigorated to see what open source game engines the Rust community was working on. By far the largest and most active project is [Bevy](https://bevyengine.org/), an open source ECS game engine with support for WebGPU and glTF. It supports all the features my hobby engine hoped to, and many more. The community is active and friendly, and there are many new features being worked on in the open. Suffice to say I was stoked, Bevy is exactly the kind of project I was hoping to find.

Compared to the Rust book, [Bevy's quickstart materials](https://bevyengine.org/learn/quick-start/getting-started/) are a bit more sparse. Fortunately, there's a [large set of examples](https://github.com/bevyengine/bevy/tree/main/examples) that covered every situation I needed, and a [cheatbook of snippets and advice](https://bevy-cheatbook.github.io/) for more complicated questions. Compared to other engines I've played with, there are a couple key differences getting started with Bevy. First, there's no editor - `bevy` is just a crate that you add as a dependency to your project. There are a few [community editors](https://bevy-cheatbook.github.io/setup/bevy-tools.html), but I haven't tried any yet. Second, Bevy follows the [Entity Component System (ECS)](https://bevyengine.org/learn/quick-start/getting-started/ecs/) paradigm. Components are Rust `structs` that implement the `Component` trait. Systems are functions that access or modify components. Entities are logical groupings of components. Systems interact with components via [queries](https://bevyengine.org/learn/quick-start/getting-started/ecs/#your-first-query).

I've been interested in the ECS pattern for awhile - back when I was working on [Grandma Green](http://localhost:4000/blog/2023/05/11/grandmagreen.html) we steered toward something that sort-of-kind-of emulated a data oriented design but ended up a little half baked. ECS alleges to improve performance by reducing cache misses - if your movement system wants to modify the position of every entity in the scene, it's easier to make sure the position components are paged together than if you had to access a parent game object comprised of many components. What I was most interested in is how ECS changes how games are written - Timothy Ford's GDC 2017 talk [*Overwatch Gameplay Architecture and Netcode*](https://www.gdcvault.com/play/1024001/-Overwatch-Gameplay-Architecture-and) (which includes a fantastic introduction to the ECS pattern) emphasizes how ECS reduced inter-system coupling and allowed the team to move iterate quickly on core game features. Additionally, the data oriented approach lends naturally to parallelization and network architecture design - data updates can be emitted as events for batching or sent down the wire as packets. 

## Boids

Whew. Finally here. I wanted to spend a few days putting together a short project on Bevy, to familiarize with the engine and get some experience writing Rust without the guidance of the book. I went with boids - it's a straightforward algorithm that involves common tasks like moving meshes around on the screen and inter-agent interaction. I used [V. Hunter Adams' post](https://vanhunteradams.com/Pico/Animal_Movement/Boids-algorithm.html) as reference. I won't describe the actual boids algorithm here, but I'll talk through some details of getting it going on Bevy.

### Entities, Components, Systems

The first task was looking at boids through the ECS lens. Boids are simple - they have a position, direction, and speed. In this implementation each boid is an *entity* described by two *components* - a triangle mesh and a 2d velocity vector. I use a [bundle](https://bevy-cheatbook.github.io/programming/bundle.html) to bind these components together as a boid.

```
// Marker for entities tracked by KDTree
#[derive(Component, Default)]
struct SpatialEntity;

#[derive(Component)]
struct Velocity(Vec2);

#[derive(Bundle)]
struct BoidBundle {
    mesh: MaterialMesh2dBundle<ColorMaterial>,
    velocity: Velocity,
}
```

There are 3 *systems* for boid motion. The *flocking system* implements boid behaviors by emitting a per-boid change in velocity. The *velocity system* listens for changes in velocity, updates each boid's velocity vector, and clamps their overall speed while steering them within screen-space boundaries. Finally, the *movement system* updates the position of each boid according to its final velocity.

```
fn flocking_system(
    boid_query: Query<(Entity, &Velocity, &Transform), With<SpatialEntity>>,
    kdtree: Res<KDTree2<SpatialEntity>>,
    mut dv_event_writer: EventWriter<DvEvent>,
) { ... }

fn velocity_system(
    mut events: EventReader<DvEvent>,
    mut boids: Query<(&mut Velocity, &mut Transform)>,
) { ... }

fn movement_system(
    mut query: Query<(&mut Velocity, &mut Transform)>,
) { ... }
```
### Boid Lookup Acceleration

Brute force boids implementations tend to be slow. Evaluating each boid against every other scales exponentially, so it's common to use a spatial lookup to accelerate comparisons. I considered implementing a simple grid lookup similar to [jfgreen's `rusty-boids`](https://github.com/jfgreen/rusty-boids), but in the spirit of the bazaar sifted through crates to see what the community had already made available. I found [laundmo's `bevy-spatial`](https://github.com/laundmo/bevy-spatial), which wrap's [Terada Yuichiro's `kd-tree`](https://github.com/u1roh/kd-tree) and provides a convenient interface for tracking transformable component's in your own Bevy application. The [distance2d example](https://github.com/laundmo/bevy-spatial/blob/main/examples/distance2d.rs) seemed to fit my use case exactly.

It really was as simple as spawning each boid with the `SpatialEntity` component (described above) -

```
commands.spawn((
    BoidBundle { ... },
    SpatialEntity
));
```

and querying the tree for each nearby boid -

```
fn flocking_system(
    kdtree: &Res<KDTree2<SpatialEntity>>,
    ...
) -> {
    ...

    for (_, entity) in kdtree.within_distance(pos, BOID_VIS_RANGE) {
        ...
    }
}
```

Nice!

### The Devil in the Details

When it came time to implement flocking, I ran into a very Rust-y issue. Consider the following piece of pseudocode:

```
for each boid (boid1):
    dv = Vec2(0, 0)

    for each other boid (boid2):
        dv += flocking_dv(boid1, boid2)
    
    boid1.velocity += dv
    boid1.position += boid1.velocity
```

Simple enough. Now let's turn those boid references into a Bevy query:

```
fn flocking_system(
    mut boids: Query<(&mut Velocity, &mut Transform)>,
) {
    for (mut v0, mut t0) in boids {
        for (v1, t1) in boids {
            ...
        }
    }
}
```

See the problem? The flocking system accesses the Velocity and Transform components for each boid through the `boids` query. We intend to update each boid's velocity and position, so we make the query mutable. The problem arrives when we want to compare this boid to each other - in order to iterate over every other boid, we have to reuse the same query object. Because we've already borrowed the query as `mutable`, there is no valid way to borrow it again.

This is a common issue -
- [stack overflow: How to do a nested query in a Bevy system?](https://stackoverflow.com/questions/63768244/how-to-do-a-nested-query-in-a-bevy-system)
- [r/bevy: Rust/Bevy noob having trouble with queries.](https://www.reddit.com/r/bevy/comments/whu1m9/rustbevy_noob_having_trouble_with_queries/)
- [r/bevy: What's the rust-ish way to solve this?](https://www.reddit.com/r/rust/comments/17om0uf/whats_the_rustish_way_to_solve_this/)

This is a good example of Rust identifying a bug - we are attempting iterate over data we are currently modifying. By updating the velocity and position for each boid as we traverse the query, future boids would make decisions based on the updated state of previously evaluated boids, making flocking behavior order dependant.

There are multiple ways to solve this, but my favorite was inspired by [KV Le's rusty-boids](https://github.com/kvietcong/rusty-boids). Emitting [`Events`](https://bevy-cheatbook.github.io/programming/events.html) seemed like the most idiomatic solution in Bevy and sets up multithreading neatly. This is why the flocking system is in two parts - the *flocking system* that emits a `DvEvent` for each boid, and a *velocity system* that listens for `DvEvents` and updates each boid's velocity.

### Multithreading

Decoupling boid iteration and velocity updates makes multithreading a breeze. Similar to KV Le's implementation, I use Bevy's [TaskPool](https://docs.rs/bevy_tasks/latest/bevy_tasks/struct.ComputeTaskPool.html) interface to dispatch the work of calculating dv's over multiple threads.

```
fn flocking_system(
    boid_query: Query<(Entity, &Velocity, &Transform), With<SpatialEntity>>,
    kdtree: Res<KDTree2<SpatialEntity>>,
    mut dv_event_writer: EventWriter<DvEvent>,
) {
    ...

    for batch in pool.scope(|s| {
        for chunk in boids.chunks(boids_per_thread) {
            // create per-thread references to move into scope 
            let kdtree = &kdtree;
            let boid_query = &boid_query;
            let camera = &camera;
            let window = &window;

            s.spawn(async move {
                // dv results for this batch
                let mut dv_batch: Vec<DvEvent> = vec![];

                for (boid, _, t0) in chunk {
                    // dv for this boid
                    dv_batch.push(DvEvent(*boid, flocking_dv(
                        kdtree, boid_query, camera, window, boid, t0,
                    )));
                }

                dv_batch
            });
        }
    }) {
        // submit the batch of dv's
        dv_event_writer.send_batch(batch);
    }
}
```
Unfortunately [multithreading is currently unavailable in the browser](https://github.com/bevyengine/bevy/issues/4078).

### Building for the Web

The final piece of this project was building it for the browser by compiling to WebAssembly. I don't have much to add here, the short [tutorial in the Bevy docs](https://bevy-cheatbook.github.io/platforms/wasm.html) worked without any issues. You can find this project hosted for the browser here: [Interactive Boids Simulation](/blog/2024/04/22/bevy-boids-interactive.html).

## The End

We finally made it! This was supposed to be a quick recap of an even quicker project, but ended up as a week-long ramble. So it goes. I'm loving Rust and plan to work with it as much as I'm able, and have some plans for getting involved with Bevy - if you're interested, I will probably share them here and maybe on my LinkedIn. Thanks for reading.