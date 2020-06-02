---
layout: post
title:  "The benefits of modular software design"
date:   2020-06-02 07:30:00 +0100
tags:  development modular-design
---

Modular software design can be briefly described as splitting a software system into multiple well-isolated, autonomous parts (modules) that are loosely coupled with each other.

## The benefits

- Replaceability. As requirements change individual modules can be easily replaced with new ones to handle the new requirements.
- Because of isolation the work can be split across multiple people/teams.
- Each piece of the system can fit in our head more easily.
- Old code can be deleted more easily and with less risk.
- Implementation details are easier to change.
  - This is useful when requirements change and new needs emerge.
  - This allows you to go for simpler implementation and tools at the beginning and gradually switch to more complicated ones when (and where) you need them, instead of paying the cost up-front.
- Caching and query optimisation is often easier with good boundaries and fine-grained data.
- Easier to test & debug (provided that you have to have clear interfaces to catch integration bugs quickly.)

## What's not on the list

- Low coupling & high cohesion. These are not the benefits - it's what makes the architecture modular
- Reusability, which often increases coupling in the long run, so it hurts modularity.
- Expressiveness. Desirable, but orthogonal to modularity.
- Good domain understanding & DDD-style code. While modularisation is one of the crucial tools in DDD, it won't help if you don't understand the domain you're working with.

## Relevant resources:

- My article about the costs of modular design: [https://mkaszubowski.com/2020/05/28/costs-of-modular-software-design.html](https://mkaszubowski.com/2020/05/28/costs-of-modular-software-design.html)
- Jessica Kerr wrote a great piece on the dangers of reusability: [https://jessitron.com/2017/02/23/reuse/](https://jessitron.com/2017/02/23/reuse/)
