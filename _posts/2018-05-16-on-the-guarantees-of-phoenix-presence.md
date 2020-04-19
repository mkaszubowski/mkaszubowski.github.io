---
layout: post
title:  "On the guarantees of Phoenix Presence"
date:   2018-05-16 21:47:57 +0100
tags: elixir distributed-systems
---

[Phoenix Presence](https://hexdocs.pm/phoenix/Phoenix.Presence.html) is a feature that allows you to track processes across the cluster. The most common use case is tracking which users are online in a distributed environment without relying on a single source of truth.

I won’t go into details how to use Presence, so if you aren’t familiar with it, you can learn more by reading this tutorial: [http://www.thegreatcodeadventure.com/tracking-user-state-with-phoenix-presence-react-and-redux/](http://www.thegreatcodeadventure.com/tracking-user-state-with-phoenix-presence-react-and-redux/)

# **Highly Available, Eventually Consistent**

Presence was designed to be as available as possible. It’s based on [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) which means, that it doesn’t require any central database, it can tolerate node failures and network partitions, while still accepting updates. This availability comes at a price, though.

According to the [CAP theorem](https://en.wikipedia.org/wiki/CAP_theorem), in any distributed system, there’s a tradeoff between availability and consistency. Because Presence favours availability, it means that it doesn’t always return strongly consistent values and you can have different values at different nodes.

Let’s see an example.

# **A Real-life Use Case**

In one of the projects, we have a chat feature. Every time a message is sent, there’s this kind of logic:

```elixir
case get_connection_pid(client_id) do
  pid when is_pid(pid) ->
    send(pid, {:new_message, message})

  nil ->
    send_push_notification(client_id, message)
end
```

`get_connection_pid` functions uses Phoenix Presence to check if a user is online. `pid` identifies a [cowboy](https://github.com/ninenines/cowboy) process responsible for the WebSocket connection. Basically, if a user has the application opened, we send a message via the WebSocket and if not — we send a push notification. So what’s wrong?

One possible execution looks like this:

![https://miro.medium.com/max/2980/1*bs7ki8b_O4RG8ICN5c3aZw.png](https://miro.medium.com/max/2980/1*bs7ki8b_O4RG8ICN5c3aZw.png)

Let’s see what’s going on here.

1. A user connects to the WebSocket on `Node A`. The connection is handled by a cowboy process with `pid_1`.
2. Somebody sends a message to the user. The request to send a message is executed by `Node B`. It sees that this user is connected to `pid_1` and sends a message there. Even though it’s on a different node, the message can be transparently sent thanks to the Erlang VM. Everything’s fine for now.
3. The user disconnects. They might have just lost the Internet connection or they might have just closed the application. This time, the communication between two nodes is slower. Information about disconnection takes more time to get to `Node B`.
4. The user connects again. He is handled by the same node, but a new connection is handled by another process with `pid_2`. Again, the communication between the nodes is slower and this message takes some time to get to `Node B`.
5. Here, things get messy. A new message arrives, and `Node B` checks if the client is online. At this point, messages from points 3 and 4 haven’t yet been received by `Node B`. **This means that this node still thinks that the user is connected using `pid_1`, which is no longer alive. Thus, the message is lost.**

This execution is quite unlikely to happen. I ran some statistics and it turns out that we lose less than 0.5% of messages this way. But there’s a catch. With about one million messages a day, 0.5% is about 5000 lost messages. That’s not good. And because it’s so rare, debugging this problem is really hard.

# **Conclusions**

It’s important to understand that I’m not saying that Phoenix Presence is bad. On the contrary — it’s an amazing tool and you should definitely use it. The point is, that **it’s important to understand how our tools work and what guarantees they give you.** The fact that something seems to work, doesn’t mean that it will always behave the way you think. Without understanding the internals, you can only have hope.

What about Presence in particular? As always, it depends. You should definitely use it when consequences of being wrong are not dangerous — for example when displaying online users in the UI. On the other hand, when this information is important for the algorithm you’re using, it might be better to use a strongly consistent source of truth (and accept weaker availability). The most important thing — distributed systems are hard. If something can go wrong, it certainly will and you should be aware of potential threats.
