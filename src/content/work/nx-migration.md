---
title: Migration to An NX Mono Repo
publishDate: 2023-03-03 00:00:00
img: /assets/nx.webp
img_alt: NX Logo
description: |
  Combining up our disparate apps into one super duper mono-repoed mega app
tags:
  - Dev
  - Mono Repo
  - NX
  - Code Quality
---

Where I've been working for several years has always had a few problems with code quality and technical debt. As a team we've always tried to address them in good time, but with business needs and deadlines, technical debt always builds up.

One of the most major problems we faced was was had 3 separate apps that handled different problems for the same users. These naturally sort of sprouted up over time, and they certainly worked, but having 3 apps for one user to use your platform made no sense, so they had to be combined into one big app.

I feared the one big app. I've worked on big monolith projects before and they are always a nightmare. Code becomes a tangled mess of interdependent nonsense and you can never confidently change anything for fear something else will come crumbling down into nothing.

Enter NX, the mono repo tool (that claims to be not a mono repo, for some reason that I don't understand).

The process for us migrating to NX was relatively straightforward and easy. We just set up an NX workspace, made all our old apps into nx libs, and made a new app that imported the libs and exposed them on their own routes. We could then sensibly refactor common components into their own libs, and reorganise as we saw fit. It was remarkably easy.

We took the oppurtunity of migrating to a new platform to also migrate from Flow Types to TypeScript. Which was reasonably straightforward (with the help of [flow-to-ts](https://github.com/Khan/flow-to-ts)). Although it take quite a lot of time to go through and fix all the types before we could continue.
