---
layout: post
title:  "Grandma Green"
date:   2023-05-11
author: Christian Robles
category: blog
image: "/assets/images/grandmagreen/garden-night.jpg"
published: true
---

<figure>
  <img src="/assets/images/grandmagreen/appstore-downloads.jpg" class="no-shadow"/>
  <figcaption>Update 8/1/23: Grandma Green has almost 7 thousand downloads on the App Store!</figcaption>
  <figcaption>---</figcaption>
</figure>

<figure>
  <img src="/assets/images/grandmagreen/garden-night.jpg" />
  <figcaption>Grandma and her cute Plant Golems.</figcaption>
</figure>
Finals are finished and graduation is just around the corner. Yesterday was the [2023 USC Games Expo](https://www.uscgamesexpo.com/), where teams and individuals from a variety of disciplines at USC shared their games and interactive art installations. I attended as a member of [Grandma Green](https://www.grandmagreen.app/), a virtual pet and gardening simulation game led by [Jebby Zhang](https://www.jebbyzhang.com/).

It's been a blast - the team is a ton of fun to work with, thanks in no small part to all of our great leads. Our wonderful Engineering Lead [Jamie Leodones](https://www.linkedin.com/in/jamie-leodones-a4800a238/) and our amazing Design Lead [Sanketh Bhat](https://www.linkedin.com/in/sanketh-bhat/) deserve a special shout-out. The constant interdisciplinary collaboration is a huge breath of fresh air. Through all the struggle of [Directed Research](/blog/2022/11/17/directed-research.html) Grandma Green has kept me excited about programming.

I joined Grandma Green last Fall as a gameplay programmer. Since then I've had my hands in most pieces of the project, with primary focus on owning the delivery of the core garden state, Mendelian trait inheritance and rotating tasks systems.

## State of the Game

<figure>
  <img src="/assets/images/grandmagreen/lets-play.jpg" />
</figure>

When I joined the project last Fall Grandma Green was already a couple months into development. The project began as Jebby's thesis and the initial team formed late last spring and began work through the summer. I joined the project through USC's Advanced Game Projects course where I got credits towards my Masters for my contributions to the game. In the AGP program, games begin in the spring of the previous year and are released for expo the following spring, typically accompanied by a release on a distribution platform like Steam or the App Store.

There were a couple technical design choices made before my joining that shaped implementation of features:
- The project is made in Unity 2022.1.4f1.
- All game state information is stored in `structs`. In C# `structs` are [value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types), meaning state can be passed between systems by value only. The [`ref struct`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct) type is generally not useful here. Additionally, [`record structs`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/record) are not available in C# 9.
- The implementation of the save system requires that important data be massageable to flat lists.
- State systems should be scene-agnostic [scriptable objects](https://docs.unity3d.com/Manual/class-ScriptableObject.html).

## Keeping Track of Plants

<figure>
  <img class="no-shadow" src="/assets/images/grandmagreen/flowers.jpg" />
</figure>

The first system I was tasked with was Garden State. This system is responsible for tracking and updating the state of plants across all of Grandma's gardens. Garden State is all about providing an interface to `PlantState`, and is managed across three subsystems.

### `PlantState`

This is the `struct` that represents the current state of a plant in a garden. A plant is described by a `PlantId`, a `Genotype`, the `cell` it occupies in the garden, and some metadata for its growth, health and fertilization.

### `GardenManager.cs`

This script is a `ScriptableObject` that provides the interface for systems to interact with garden state. It maintains a `list` of all gardens, where each garden is a `Dictionary` that maps a `cell` to a `PlantState`. Its interface provides CRUD functions that allow Grandma and her Golems to plant and harvest plants. It is also responsible for tracking the growth of plants over time, and emits events for plant growth, wilting, and death.

### `GardenSaver.cs`

This script is a `ScriptableObject` that shims Dictionary functionality between the `GardenManager` and the save system. Previous constraints require that data-to-be-saved be stored in homogenous lists that derive from a provided `ObjectSaver` class. A player may have many plants across a handful of gardens, so it's important to minimize searching through plant data when an entity interacts with a plant in a garden. This shim provides constant-time lookup for `PlantState` instances by storing a list of `PlantState` for each garden and a list of `cells` (`Vector3Int`) in the save system. On startup, it builds a [`Dictionary<cell,PlantState>`](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2?view=net-7.0) that the `GardenManager` can use to look up `PlantState` by `cell`.

### `GardenAreaController.cs`

This script is the [`monobehavior`](https://docs.unity3d.com/ScriptReference/MonoBehaviour.html) that lives in the garden game scene. It consumes data from `GardenManager` and is responsible for making sure the correct sprites and particle effects are displayed in the garden, as well as making sure the correct tile (e.g. grass, dirt, watered dirt) is displayed. On scene entry it looks over all `PlantState` in this garden and displays the correct sprite. On entity interaction it requests updates to `GardenManager`. It subscribes to the plant growth, wilting and death events emitted by `GardenManager` for this and updates the corresponding sprite & particle effects for that plant.

## Mendelian Inheritance

<figure>
  <img src="/assets/images/grandmagreen/cultivision.jpg" />
  <figcaption>Inheritance Punnett square in the Cultivision interface</figcaption>
</figure>

In Grandma Green plants can be crossbred with a simplified Mendelian inheritance system. Every plant has a `genotype` that encodes two `traits`. The first trait `[Aa]` for all plants is the size of the plant, where `aa` is medium, `Aa` is medium, and `AA` is large. The second trait `[Bb]` is unique to each plant type - for flowers it is color, for fruits it is variety, for vegetables it is the length. When a plant is harvested one of its cardinal neighbors is randomly selected for crossbreeding, then a random coordinate on the  square is selected that determines the genotype of the harvested seeds.

Through crossbreeding the player can generate special `Mega` traits. Megas are created by crossbreeding plants with duplicate homozygous secondary traits, i.e. `[BB] X [BB]` or `[bb] X [bb]`. By breeding these duplicate traits for two successive generations, players generate special `Mega` variations that are rare colors, varieties, or extreme sizes that can be sold in the town square for extra money or submitted to contests for an extra edge.

The `Genotype` struct encapsulates all of the behavior to support crossbreeding. The traits are stored as enums - `Size { Small, Medium, Large }` and `Trait { Dominant, Heterozygous, Recessive }`. To simplify crossbreeding, I recognized that Punnett square construction is the same for every pair of genotypes regardless of the input traits. I perform crossbreeding by concatenating the string representation of two genotypes, e.g. `AaBbAAbb`, and I select a random index in a baked mapping that represents the Punnett square -

```
int[][] punnettSquare = new int[][] {
  new int[] { 0, 4, 2, 6 }, new int[] { 0, 4, 3, 6 }, new int[] { 1, 4, 2, 6 }, new int[] { 1, 4, 3, 6 },
  new int[] { 0, 4, 2, 7 }, new int[] { 0, 4, 3, 7 }, new int[] { 1, 4, 2, 7 }, new int[] { 1, 4, 3, 7 },
  new int[] { 0, 5, 2, 6 }, new int[] { 0, 5, 3, 6 }, new int[] { 1, 5, 2, 6 }, new int[] { 1, 5, 3, 6 },
  new int[] { 0, 5, 2, 7 }, new int[] { 0, 5, 3, 7 }, new int[] { 1, 5, 2, 7 }, new int[] { 1, 5, 3, 7 }
};
```

Generational `Mega` tracking is achieved with an enum `Generation { P1, F1, F2 }` where P1 is an unbred plant, and successive breeding of duplicate homozygous eventually reaches the `F2` Mega stage. There's not much interesting logic here, just checking if traits are identical and comparing generations.

## Rotating Tasks (Dailies/Weeklies)

<figure>
  <img src="/assets/images/grandmagreen/bulletin-board.jpg" />
  <figcaption>Daily tasks on the Bulletin Board</figcaption>
</figure>

Grandma Green has daily/weekly tasks that reward the player with money, seeds or garden decorations when completed. Tasks are tracked on the Bulletin Board in the town square, and ask the player to perform common actions like watering plants, tilling grass or selling harvested plants on the market.

The task system is composed from two components -

### `BulletinEventListener.cs`

The `BulletinEventListener` is an abstract class that describes an interface for a task that subscribes to certain game events, exposes the player's progress towards completion, and raises an event when a task is completed. Each implementation provides custom event handling behavior and is responsible for reporting a completion value that can be parsed into the UI progress bar. For example, the "Hydration Station" task subscribes to the "plant watered" event and increments a counter towards its target total. When it reaches the total, it marks itself as complete and raises its completion event.

### `BulletinDataStore.cs`

The `BulletinDataStore` is the scriptable object that maintains a list of all potential tasks. a `Task` has a `BulletinEventListener` that it uses to track activity towards completion, as well as some tuneable difficulty parameters (how many plants to water) and other information like flavor text and the friendly name to show in UI. Previously completed tasks are marked as completed until the week changes, at which point all tasks progress is reset and marked as incomplete.

Bulletin integration into the save system is straightforward - all task state is pickled on save, and unpickled on game session start.

## Closing Thoughts

I had a blast working on Grandma Green. The whole team was a ton of fun and I'm going to miss everyone a lot. 🥲 This was my first significant foray into game development, and I'm kind of hooked. Here's hoping for more great game projects in the future!
