---
layout: post
title:  "Events and different kinds of coupling"
date:   2019-06-28 21:47:57 +0100
tags: development design
---

There’s a common misconception that events reduce coupling. Sure, this statement can be true. But it’s extremely inaccurate.

The problem is that “events” has become an umbrella term, which combines different concepts. When talking about events, we talk about:

- Domain events, which encapsulate business logic
- Asynchronous processing
- Pub-Sub mechanisms, message queues
- Consistency issues
- Traceability (think: Event Sourcing)

How does all of this relate to coupling?

# **Coupling**

Coupling is also an umbrella term. There is no single type of coupling. Not every coupling is bad. Some types of coupling are necessary and unavoidable. Some of the different forms of coupling include:

- **Logical/Development coupling** — the most common form, visible in the source code; if the code of one thing changes, some other thing has to be changed as well
- **Operational/Temporal coupling** — related to time; one thing requires the other to be running at the same time, or: calls between multiple modules have to be done in a certain order
- **Platform/Infrastructure coupling** — multiple things relying on the same dependency, platform, or a piece of infrastructure
- **Semantic coupling** — multiple things relying on the same shared concept, eg. domain model
- **Functional coupling** — multiple things sharing the same responsibility

# **How events impact coupling**

When using (or talking about) events, it’s important to think what kinds of coupling you want to fight.

Operational coupling can be reduced by asynchronous event handling. If you’re using microservices architecture, it might be crucial. But if your system is a monolith, operational coupling might not be an issue for you. Since asynchronous processing comes with its own risks, maybe it’s better to stick to synchronous operations if possible.

Even if you need asynchronous processing, you don’t need any PubSub mechanism. If you only have one producer and one consumer, there’s probably nothing wrong with smart consumer polling for data. It allows you to care less about message delivery guarantees and reduces platform coupling.

If your main problem is logical coupling inside a single deployment unit, you might use event handling to decouple modules at the code level, but this handling can be synchronous and transactional. You can decouple things logically, while still relying on all the guarantees and simplicity of ACID transactions.

Event handling tends to increase semantic coupling. Each event represents some domain concept. The more modules depend on this concept, the more stable it becomes and the harder it is to change it.

Functional coupling is rarely reduced by events. Take a classic e-commerce system. A user has to pay for an order before this order is shipped. If you want to change that process (for example: for VIP customers an order can be shipped before paying), that change is equally hard if you communicate modules by asynchronous events, direct function calls, or synchronous requests to another service.

# **Conclusions**

Don’t reach for external message queue if you only need to make the code easier to understand. Don’t reach for message queue when simple polling will do. Don’t reach for events in any form just for the sake of “reducing coupling”.

Firstly, understand the problems you have. Secondly, get to know the tools, but analyze what problems the tool solves. Thirdly, use the simplest tool that solves your problem.

There’s nothing wrong with keeping potential problems in mind and preparing the system for future changes, but try to postpone making any important decisions to the last responsible moment.
