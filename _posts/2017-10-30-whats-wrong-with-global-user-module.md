---
layout: post
title:  "What’s wrong with a global User module?"
date:   2017-10-30 21:47:57 +0100
tags: elixir modular-design
---

This post is inspired by a code review I’ve done at work today. We generally try
to build our applications as a set of loosely coupled components, so when I saw
a `User` schema in the root project directory, it got me thinking about its
impact on the overall architecture of the system.

My first thought was that we, as web developers, are too used to thinking in
terms of *nouns* instead of *verbs*. We think about database tables, records,
associations. This may work fine for simple CRUD resources, but not for entire
application. For more complex features, we should really start to think about
the process. What are the features of the application? What can the end user do
with it? This is what’s important. All the nouns are, in my opinion, just an
implementation detail (do you think your consumer care about what database you
use or how many tables you have?).

So what’s wrong with having a global `User` module?

In my opinion, it’s often added without any thought. It’s obvious that every
application should have a `User` entity, right? But I think that doing so often
leads to a bad design. I think it encourages you to just add fields when it’s
necessary, instead of thinking about proper architecture and creating well
separated modules. I’m pretty sure that once you create a global `User` schema,
it will grow a lot and will be coupled with *almost all* parts of the system.
This means that you’ll have no flexibility and the entire system will be really
hard to understand. Have a look at the `User` modules/classes in your current
projects. Is this code easy to understand? Are you comfortable when you change
it?

Scientists show that we are capable of keeping only limited number things in our
mind at any given time, so it’s extremely important to be able to think about
different parts of the system in isolation. In order to do that, each part have
to be as small and as loosely coupled with other parts as possible. And this is
not possible when you have a huge `User` module which couples all the parts
together.

# **A better way**

On the other hand, let’s imagine that you have `Authentication` component.
Inside it, you would probably have some `User` schema with the fields necessary
for the authentication process. This schema would be *private* for this
component. It shouldn’t ever be used by other components, so it wouldn’t be
possible to add anything that’s not related with authentication. This guarantees
that it would remain small, focused on doing one thing well and disconnected
from the rest of the system. This would make it really easy to understand and
change.

So if you wanted to, for example, add profile information, you couldn’t just add
fields like `name`, `email` to the existing user schema. You’d have to create a
separate component for this feature with a *separate database table.* This would
result with two really small parts (`Authentication`and `Profiles`) which are
connected only by some user identifier. You can also use that identifier to
reference the database tables to ensure data integrity if you want to. But on
the application level, they would be a separate things.

If you run an e-commerce site, you would probably have some shipping
information. So you’d create a `Shipping` context and keep the necessary
information there.

Every time you wanted to add a new feature, you would just create a small
component instead of adding fields to a huge `User` with all the
information. Authentication, Reviews, Profiles, Payments, Shipping, … - they all
are somehow connected to the user (as a person who uses the application), but
they are all interested in different information and this information shouldn’t
be kept in one place because it’s really hard to understand and change later
(I’m talking about “one place” on the application level. As mentioned before,
you might want to keep all the data in the same place - one database for
example - to enforce the data integrity).

For comparison, with global `User` schema, it would probably already have all
the fields connected with authentication, profiles, shipping etc. Now, imagine
that you want to change one part of your system. If your `User` is public and
every part of the system depends on it, how can you be sure that
removing/changing one field won’t affect other parts? You can’t reason about one
component of the system in isolation, you have to keep everything in your mind
and this is almost impossible.

You are less flexible, too. Think about shipping. Imagine that you have to add
an ability to have multiple shipping addresses associated with one user. If you
have that information stored in the `User` struct, you have a lot of work to do.
You have to create a separate schema/table, migrate the existing data, modify
the `users` table in the database and make sure that you don’t break anything
while doing that. On the other hand, when you start with seperate context with a
function which returns the address, `Shipping.fetch_shipping_info(user_id)`, you
just have to return the list of addresses instead of one. There’s no data
migrations necessary, the impact on the entire system is a lot lower, so there’s
less risk involved.

That’s why I was talking about internal data structures as *private*. Other
parts of the system are not allowed to use them. The only way to access the
information is by using a clearly defined APIs.

Now let’s look at benefits from the inside and from the outside of the
component.

# **Inside the component**

Inside the `Profiles` component you may have
a`Profiles.fetch_profile(user_id)` function that returns a map or struct
with `:name`, `:email`, `:phone_number` fields. But does it mean that you have
to have a schema and database table with this exact columns? Not at all!

Inside the `Profiles` context, you can make all the changes you want as long as
you return `:name`, `:email`, `:phone_number` fields from
the `Profiles.fetch_profile(user_id)` function.

You want to change the internal database schema? Or even change the database? Or
drop the database and keep everything in memory? Or move the profiles feature to
a separate service and retrieve the information via a HTTP call? As long as you
maintain the interface, you’re good to go.

What’s important, that you can make all this changes and be *sure that you won’t
break anything* in other parts of the system because you hide the implementation
details behind the interface defined by `Profiles.fetch_profile(user_id)`.

# **Outside of the component**

From the outside, you don’t have to care about the implementation details to
understand what this component does. You only care what the interface is. You
provide `user_id` and get the information you need. It’s simple. It’s easy to
understand. You don’t have too keep all the details in mind. You can understand
the system better because it consists of loosely connected parts which
communicate by usingclearly defined interfaces instead of details like schemas
and database tables.

# **Testing**

There’s another benefit of having the system built as a system of loosely
coupled components. It is so easy to write the unit tests for the individual
pieces of the system. You don’t have to worry about complicated setup because
the interface of the component is really small and the component is isolated
from the rest.

Your tests also become less fragile. Because you have a clean interface used to
interact with the component, you can use this interface to test it. This is
really a big deal because it allows you to refactor your code with confidence.
How many times did you refactored your code and had to change **both tests and
the application code**? This was because your tests were testing the
implementation, not the behaviour of the component. The real power of
refactoring is that you can change the tests **or** the application code while
the other part remains stable. Only this gives you the confidence that you don’t
break anything. If your tests passed before the change in the code and are still
passing after, it means that everything is fine. If you are changing your tests
and they still pass after the change, while the code is stable, you’re safe too.
If you change both at the same time, you cannot have any confidence.

Think about this: tests are just another client of your application code. This
means that they, too, should be decoupled from too much implementation details.
They should be thought out and designed properly. Only then they can give you
the confidence that you count for. Fortunately, having a good design of the
application code often improves the design of the tests.

# **So why we choose nouns?**

Coming back to the beginning — when modelling the system, let’s start to think
about processes, features and actions instead of data structures or database
tables.

But why do we often start with nouns? This may be caused by the misunderstanding
of the Object Oriented programming which was, for a long time, the most popular
programming paradigm. We was really focused on *objects* and missed the key
idea. In the words of Alan Key, the father of object orientation:

> Smalltalk is not only NOT its syntax or the class library, it is not even
> about classes. I’m sorry that I long ago coined the term “objects” for this
> topic because it gets many people to focus on the lesser idea.The big idea is
> “messaging” — that is what the kernal of Smalltalk/Squeak is all about.
> (…) The key in making great and growable systems is much more to design how
> its modules communicate rather than what their internal properties and
> behaviors should be.

[(http://lists.squeakfoundation.org/pipermail/squeak-dev/1998-October/017019.html)](http://lists.squeakfoundation.org/pipermail/squeak-dev/1998-October/017019.html)

So don’t start with the properties. Even though there’s some real person in
front of the device, this doesn’t mean that you should have some `User` entity
with all of his/her properties. Think about how the components of the system
should communicate to accomplish any given task. Only then figure out what data
structures are necessary to implement this.

# **Common problems**

Of course designing the system this way requires more initial work. It seems
easier to just dump everything to the `User` schema. But remember that it’s only
easy because when adding new features, you just think about one thing at a time.

It gets harder when you have to change things. To make the change you have to
create the mental model of the entire system, understand the connections, wonder
if anything will break. And unfortunately, most of our work is changing existing
systems, not adding new features. So with a little more initial work, you’re be
able to make your life a lot easier in the future. You’ll thank yourself later.

Another thing that may worry you is performance. You might think that splitting
the system into isolated component would be slower because you would often have
to make more queries to the database to get the necessary information (instead
of just fetching the entire `User`). And while this statement might be true, the
truth is that most of the time it doesn’t matter at all. Computers and database
are really fast these days and they really can handle it. You might want to
optimise things later, but as always - premature optimisation is the root of all
evil. As someone smart once said: “First make it work, then make it elegant.
Only then, if necessary, make it fast, while keeping it elegant”. So **optimise
only when it’s absolutely necessary** and you have the data to prove that it is.

# **Summary**

Think about verbs instead of nouns. Model the system around features, not
data. Think before you start writing code. Creating software is a marathon, not
a sprint. Don’t be afraid to sacrifice initial productivity for a better
architecture. It will help you in the log run.
