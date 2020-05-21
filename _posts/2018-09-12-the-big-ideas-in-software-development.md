---
layout: post
title:  "The big ideas in software development"
date:   2018-09-12 21:47:57 +0100
tags: development design
---

If you look at the history of Computer Science, you can see that there are few
big ideas that keep emerging. Is there a lesson that we may learn from this?

# **Will Serverless Change the World?**

Serverless computing is the new hot thing right now. It’s gaining a lot of
attention. Some people believe that it will change the way we write software.

Serverless is based on the idea of Function as a Service (FaaS). In this
approach, you write your code as a set of short-lived functions, which are
invoked after some event occurs. That’s it. You don’t have to care about how the
code is actually invoked, you don’t have to care about where the code lives. You
write the code and the runtime takes care of everything else. You get high
availability, scalability, fault tolerance for free. That must be a novel idea,
isn’t it?

# **The Other End Of Spectrum**

Let’s move back 30 years, when Ericsson was developing software to run on
telephone switches. Due to the nature of those systems, they have to be
extremely reliable and fault tolerant. They often run for many years without
restarting. That’s how Erlang was born.

So if you use Erlang (or Elixir), you might be skeptical about the Cloud and
Serverless Computing movements. They favor using short-lived containers or even
shorter-lived functions. Are you doing something wrong when using a platform
that is designed with long-running systems in mind?

Do you know what is the most basic concept of Erlang/Elixir concurrency? It’s a
process. Every piece of code you run in the Erlang VM is executed inside a
process. The processes are really lightweight so you can spawn thousands or
millions of them at the same time. They are also completely isolated from each
other and share no memory. If you want to coordinate two processes, you have to
do this by asynchronous message passing. The Erlang VM runs its own scheduler
which controls when each process can execute its code.

Does it sound familiar?

An Erlang process runs its code in reaction to messages being sent to this
process. The Erlang scheduler decides when each process is executed.

A function in FaaS is executed in reaction to some event. The runtime decides
when each function executed.

# **The Idea**

You see, FaaS at its core is based on the same idea as Erlang. This idea is
messaging and isolation. Both Erlang and Serverless are based on isolated
entities, that communicate only through messages. If you read the early works on
Object Orientation, you will find the same idea there. You will find the same
idea of asynchronous messages in Distributed Systems theory. Microservices
architecture is nothing else that isolated entities, communicating via messages.
The Unix philosophy is all about running small, independent programs, which
communicate only via input/output. Can you see the similarities here? Even
though hidden behind implementation details and marketing, there is the same big
idea at a core of all these technologies and concepts. If you study enough
success stories and analyze the trends that are emerging, you will see that the
ones that fully embrace this idea, are the most successful. Erlang continues to
be successful in highly scalable and fault-tolerant systems, even without a huge
number of people working on those systems and without the huge amount of money
put into operations
(See [Whatsapp](https://www.wired.com/2015/09/whatsapp-serves-900-million-users-50-engineers/) and [Bleacher
Raport](https://www.techworld.com/apps-wearables/how-elixir-helped-bleacher-report-handle-8x-more-traffic-3653957/)).

The companies which succeed in creating microservice-based systems are the ones
that embrace asynchronous messaging and independent entities. These are the
pillars for loose coupling, failures isolation, and simplicity of those systems.

On the other hand, people are starting to recognize that using OO languages
often leads to unmaintainable, overly complex and error-prone code. Developers
often forget, or don’t see that the core idea behind OO is messaging. Instead,
they focus on classes, inheritance, and overcomplicated abstractions.

This also means that if you learn a technology that makes this idea a
first-class citizen, you are more likely to succeed in other technologies.
Learning Erlang will make you a better OO programmer by embracing the concepts
of isolation and messaging. It will make you design better microservice
architectures. It will allow you to prevent your Serverless application from
becoming a huge, entangled mess.

Learning about those core concepts will make you realize that Serverless
computing will not change the way we write software. You will see that it is
just a cost-optimization technique. In some cases this optimization will work,
but sometimes — [not
really](https://medium.com/coryodaniel/from-erverless-to-elixir-48752db4d7bc).

# **Conclusion**

Trends in Computer Science come and go, but the core ideas stay the same for
decades. Why? Because, when you think about it, they embrace the way that the
real world works. We, as humans, are nothing more than fully isolated processes,
which can only interact with the world by sending and receiving messages. We
have worked that way for thousands of years.

If you want to be a better programmer, don’t focus so much on specific
technologies, but learn the underlying concepts instead. Learn about isolation,
asynchronous messaging, separation of concerns, composition, the power of good
and simple abstractions.

Learn about the history of programming, about the trends emerging over the last
few decades. When you analyze the ones that succeeded and the ones that failed,
you will see a lot of similarities. You will see the same few ideas in the
successful ones. You will be able to apply the same ideas in any new technology.
It will make you a better programmer.
