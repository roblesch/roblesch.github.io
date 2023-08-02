---
layout: post
title:  "Grandma Green"
date:   2023-05-11
author: Christian Robles
category: blog
published: true
---

<figure>
  <img src="/assets/images/grandmagreen/appstore-downloads.jpg" class="no-shadow"/>
  <figcaption>Update 8/1: Grandma Green has almost 7 thousand downloads on the App Store!</figcaption>
  <figcaption>---</figcaption>
</figure>

<figure>
  <img src="/assets/images/grandmagreen/garden-night.jpg" width="60%" />
  <figcaption>Grandma and her cute Plant Golems.</figcaption>
</figure>
Finals are finished and graduation is just around the corner. Yesterday was the [2023 USC Games Expo](https://www.uscgamesexpo.com/), where teams and individuals from a variety of disciplines at USC shared their games and interactive art installations. I attended as a member of [Grandma Green](https://www.grandmagreen.app/), a virtual pet and gardening simulation game led by [Jebby Zhang](https://www.jebbyzhang.com/).

It's been a blast - the team is a ton of fun to work with, thanks in no small part to all of our great leads. Our wonderful Engineering Lead [Jamie Leodones](https://www.linkedin.com/in/jamie-leodones-a4800a238/) and our amazing Design Lead [Sanketh Bhat](https://www.linkedin.com/in/sanketh-bhat/) deserve a special shout-out. The constant interdisciplinary collaboration is a huge breath of fresh air. For all the life [Directed Research](/blog/2022/11/17/directed-research.html) has drained from me, Grandma Green has kept me excited about programming.

I joined Grandma Green last Fall as a gameplay programmer. Since then I've had my hands in most pieces of the project, with primary focus on the technical design and implementation of the core garden state, Mendelian plant reproduction, rotating tasks, and collections (type/sprite resolution) systems. In this post I'll do a quick run-through of those systems, with some takeaways about how I might get there quicker in the future.

### State of the Game

When I joined the project last Fall the game was already a couple months into development. The project began as Jebby's thesis and the initial team formed late last spring and began work through the summer. I joined the project through USC's Advanced Game Projects course where I got credits towards my Masters for my contributions to the game. In the AGP program, games begin in the spring of the previous year and are released for expo the following spring, typically accompanied by a release on a distribution platform like Steam or the App Store.


