---
title: Programming with Pasta
publishDate: 2023-05-05 00:00:00
img: /assets/pasta.jpg
img_alt: Pasta
description: |
  Strained pasta metaphors, boiled for too long
tags:
  - SOLID
  - Principles
  - Programming
  - Base ideas
---

# Programming with Pasta

Strap in for some tortured metaphors and strained puns. Its pasta programming.

In this document we will be discussing the dangers of spaghetti, lasagna and tortellini.

## Spaghetti code

Spaghetti code is the most often used pasta metaphor in programming. Spaghetti code is difficult to follow, higgledy piggledy and probably violates most of the [SOLID](https://en.wikipedia.org/wiki/SOLID) principles. I won't talk any more about it, as its not really somehting that happens deliberately and it is something that can be avoided by following the SOLID principles, and other coding standards.

## Lasagna code

Oh no wait... you've followed all the SOLID principles to the letter, you now have models, views, controllers, view models, decorators, service objects, presenters, serialisers, and more. Everything is well organised, but it has become very difficult to read, understand and alter your code. What has happened?

You have created a code lasagna.

To see how anything works, a programmer now has to read the controller, look at the model, see the model has a service object passed into it at instantiation, look at the decorator that is decorating the model, and look at the serialiser that is outputting a representation of the model to a view.

Its the layers that you have to browse through that make the lasagna metaphor work.

The trouble with lasagna, is its very easy to accidentally write, when you are following all the rules and honestly, it isn't always a bad thing. Large codebases often benefit from being separated into many small files, and classes.

Sometimes when writing, its important to really think about the next person reading the code (that person might be "future you"), and try and understand what they will see when they read it.

There are some guidelines I would suggest you often follow:

1. If in doubt, defer to the SOLID principles, and embrace the lasagna
2. If you think "this class is far too small" it probably is, continue writing until you have finished the feature. If the class is still too small, consisder at that point inlining it somewhere else.
3. If you think "this class is far too big", it probably is, and you are probably violating the S in SOLID. Go back to point 1.

## Tortellini code

Probably more accurately called Component based architecture. Every piece of a program is an entirely contained bubble of code, which can survive on its own, or be composed together with other pasta packets, in an enticing sauce (source) to make a program.

Tortellini code is brilliant, and what we strive towards in the front end.

Component based architecture keeps everything simple, all the time, as a developer reading code can see everything they need to make decisions about how to edit or add to any component.

Problems occur when you overcook the tortellini.

This means that the pasta packets burst, and start getting little chunks of meat and cheese sticking to the outside of other pasta packets, it mixes the internals of the tortellini directly with the sauce, and that can throw off the flavour blend.

I think I've taken the metaphor too far.

So the problems occur with component based architecture, when, through ambition, time contstraints, or laziness, you open up a component, and create a hard coupling with another component. Or have a component that relies on or mutates a global state.

Essentially the downfall of the tortellini is not respecting the SOLID principles.

## Conclusion

What I'm really trying to say, is that the SOLID principles are extremely important. And even though we don't, in the front end, practice OOP, we tend towards component architecture, the SOLID principles are still relevant.

Particular care should be taken to make sure each component is a SOLID chunk.

A component should:

- Only do one thing (**S**ingle Responsibility Principle)
- Have a reasonably consistent interface (**O**pen-closed Principle)
- Have the component compose other components without requiring outside knowledge (**L**iskov Substitution Principle)
- Just be rendered and _do things_ without the thing rendering having to know about how it works (**I**nterface Segregation Principle)
- Be able to be composed together and compose other components (**D**ependecy Inversion Principle)
