---
layout: post
title:  "The costs of modular software design"
date:   2020-05-28 06:30:00 +0100
tags:  development modular-design
---

Modular software design can be briefly described as splitting a software system into multiple well-isolated, autonomous parts (modules) that are loosely coupled with each other.

Modular software design is a gold standard of maintainable software design. But to use it properly, you should know its costs and trade-offs:

## The costs:

- It requires more up-front investment.
- It requires more domain knowledge to get right.
- The consequences of getting the boundaries wrong are bigger.
- Changing a flow that spans across multiple modules is safer, but slower, since you have to change multiple modules and their contract/interfaces. This is not desirable when prototyping.
- Debugging on the early stages might be harder, since most early bugs will be integration bugs.
- Integration tests might be harder/slower to write.
- Choosing an interface/contract for each module requires some experience.
- Fully decoupling different modules from a shared database means loosing referential integrity and not being able to use DB transactions.
- Modularization and low coupling makes it easier to understand each module in isolation, but it may be harder to understand the entire system.
- Handling cross-cutting concerns (e.g. observability, privacy policies, authorization) may be harder and more mundane (and the code more scattered across the system.)

## Loose final remarks

- You can find my list of the benefits [here](https://mkaszubowski.com/2020/06/02/modular-software-design-benefits.html)
- Some of the costs may actually be marginal or even desirable.
- [Dan North](https://twitter.com/tastapod) introduced a technique called ["Spike and stabilize"](https://youtu.be/USc-yLHXNUg?t=948), which can help to mitigate some of those costs and tradeoffs (starts at 15:50). Start with a code that's not modular to figure things out and then modularize if you intend to stick with the code.
