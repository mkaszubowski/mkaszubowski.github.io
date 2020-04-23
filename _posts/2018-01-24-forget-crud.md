---
layout: post
title:  "Forget CRUD, let's focus on the meaning"
date:   2018-01-24 18:18:27 +0200
tags: development design
---

Lately, I’ve been trying to use [Command Query Separation (CQS)](https://martinfowler.com/bliki/CommandQuerySeparation.html) technique for a few days. In case you haven’t heard of this, CQS, basically, says that you can have two types of operations that clients can run in your system: **commands** and **queries**. There are two rules:

If you return the state, you cannot mutate it (queries)

If you mutate the state of the application, you cannot return it (commands)

This means that all of your operations have only one job: either return the state (or part of it) or mutate it. This approach allows to create a code that it’s easier to understand because the responsibilities are clearly separated.

It’s important to notice that *client* doesn’t necessarily means the user of the application. It might be an external system using your API. Or, maybe you, the same as me, think that the web is just a delivery mechanism and shouldn’t be coupled with the domain logic. Then, the web framework you use can be a client of your application.

With this knowledge in mind, we can see that CQS doesn’t say that one API endpoint shouldn’t both mutate and return state because you can issue both command and query during one HTTP request.

# **What about upserts?**

After introducing CQS on our Slack channel at [AppUnite](https://appunite.com/), I have been asked a really good question:

> What about upserts? With CQS, I would first have to read something, then modify it, and then, read it again.

Fortunately, the question came with an example:

> Let’s imagine a mechanism like OAuth. A client wants to receive data it can use to authenticate or authorise itself in some service. A client sends some information and the process looks like this:1. Try to fetch the user record from DB based on the provided information.2. If the record doesn’t exist — create a record based on the provided information.3. Fetch the user record (because the previous operation didn’t return anything) and return the authentication data to the client.

Surely, this doesn’t make much sense. So, what’s the issue?

# **Operations have meaning**

On the web, we are so used to CRUD. *Create, Read, Update, Delete*.

It is in line with REST verbs (`POST, GET, PUT/PATCH, DELETE`).

It is in line with SQL (`INSERT, SELECT, UPDATE, DELETE`).

Nice and clear, isn’t it?

This is the language of our tools. This is also the language that we, programmers, are so used to speaking in. There’s a big problem though. **Clients don’t care and neither do your users.**

Users want to `send a message`, not `create a message record`. **Users want to `leave a group`, not `delete a group_membership`. Users want to `add another item to the cart`, not `update a cart_item to change the quantity`.

Users don’t care about the structure of the data. They don’t care about CRUD, SQL, REST, relations or join tables. **Users just want to do things.**

# **Back to the example:**

So what about the example above? What does a client really want?

A client doesn’t want to create the user record in the DB. A client wants to get some data that allows them to authenticate/authorise themselves. From a client’s perspective, there’s no upsert operation because a client doesn’t care about the implementation, it just wants a response.

When executed again, this request would return the same data (unless the expiration time is exceeded). The conclusion: from the client’s perspective, this is a **query**. The fact that something is inserted into the database (so the state is mutated) is just an implementation detail which means absolutely nothing for a client.

# **What about the REST and SQL?**

They are just tools. Really nice tools, of course, but just tools. From the perspective of the database, it might make a perfect sense to use CRUD operations. From the REST API perspective, it might make a perfect sense to use CRUD operations.

Web framework is not your application. Database is not your application. From the perspective of your application, it probably makes no sense to use just CRUD operations, so there’s no point in limiting yourself.

# **Conclusion**

Instead of starting with CRUD, think about what the clients want to do with your application.

This will allow you to create a code which is easier to understand. You will be able to communicate more effectively with your users and non-technical people alike.

You won’t be confined to the implementation details of your database, web framework or any other tool. This means, that if you have to switch these tools, your job will be much easier and maybe you won’t be urged to leave IT and never look at the computer again.
