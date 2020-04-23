---
layout: post
title: "On restoring process state in Elixir"
date: 2017-09-02 18:18:27 +0200
tags: elixir development
---

When dealing with failures in Elixir (or Erlang), it's common to use OTP supervisors and restarts
instead of coding defensively and try to predict all that may go wrong.
But at the beggining, one might ask a question: *"Ok, restarting the process is cool,
but what about its state? I don't want to loose it!"* And it's a perfectly
valid question.

After all, one of the
[the reasons to use processes](https://medium.com/elixirlabs/when-to-use-processes-in-elixir-18287da73d47)
is to keep some application state in it. If that state is important and cannot
be recreated (e.g. data from the users), we want to be sure that it's not lost
after the failure. Loosing the state permanently might be unacceptable.


At this point it's quite natural to come up with a solution: let's just add
another process which is responsible for keeping the state from before the failure.
We can just use [`Process.flag(:trap_exit, true)`](https://hexdocs.pm/elixir/Process.html#flag/2)
and send the current state to the other process in the [`terminate/2`](https://hexdocs.pm/elixir/GenServer.html#c:terminate/2) callback.
Then, after the restart, we can load it in the [`init/1`](https://hexdocs.pm/elixir/GenServer.html#c:init/1):


```elixir
defmodule MyApp.Server do
  use GenServer

  # ...

  def init(_) do
    Process.flag(:trap_exit, true)
    state = MyApp.Backup.fetch_state()
    {:ok, state}
  end

  # ...

  def terminate(_reason, state) do
    MyApp.Backup.store_state(state)
  end
end
```

## So what's wrong?

I saw this solution a few times now (I think it was even mentioned in Dave Thomas'
["Programming Elixir"](https://pragprog.com/book/elixir/programming-elixir) book).
To be honest, I implemented it myself once in the project
I was working on a few months ago. But after taking a step back to rething this
approach, I just reverted all the changes. Let me tell you why.


In my opinion this approach misses an important point of the *"Let it
crash!"* philosophy. After all, the idea behind restarting the process is to
revert it to a **correct** state. When restoring the state from before the
crash, how can we be sure that the state was correct? This is making the process
behaviour less predictable. In my experience, the incorrect state is often
the reason of the failure. In such cases, restoring the
state may make the process crash again and again.

OTP has a way to prevent such infinite failures (each supervisor will try
to restart the process only [limited number of times before terminating itself](https://hexdocs.pm/elixir/Supervisor.Spec.html#supervise/2))
but it's not perfect. The limit may be set too high to trigger the supervision
termination. If your supervisor
can tolerate 5 failures in 3 seconds (which is the default) and your process
only received the message every 5 seconds, it will just keep failing forever.
Another scenario is that the crashes might happen randomly and setting the correct
limit might be hard or even impossible.

And even if the supervisor terminates eventually, all its children will be
terminated. For example: consider using a supervisor with
10 child processes. Even though 9 of them are working perfectly fine, the failure of
one of them might result in terminating all 10 processes. And we were told that
processes should keep the failures isolated! One failing process should never
make another one crash if they don't depend on each other.

## Alternatives

After all, we really don't want to lose the state of
the process every time a failure occurs. So what to use instead? I think there are a few solutions.

I think the best way is to use processes the way they are supposed to be used:
to **isolate failures**. Instead of restoring the state after the crash, let's
make the crash less possible. If you think that some operation might crash, just
spin off another process (e.g. a `Task`) and send the original process the
results if they're available. Then, if the process crashes in spite of this isolation, it's probably
safe to assume that the process state is invalid and let the supervisor handle the failure.

Another way is often forgotten. We heard "let it crash!" so often that we might
forget that there's another way. Sometime we can just go back to being more defensive
and try to prevent the crash (checking if something is not `nil` is the simplest example).
While it's not often used, simple `try` and `rescue` can sometimes be really helpful
(especially when using external libraries we don't control).

Of course you might still want to restore the state after the restart
(caused be a crash of the process or after the restart of the entire VM). If it's the case,
I'd go with storing the state in a permanent storage after every transition
instead of saving it in the `terminate/2` callback. There's plenty of options for the storage:
you can just use plain old database, [dets](http://erlang.org/doc/man/dets.html)
or go with [event sourcing](https://martinfowler.com/eaaDev/EventSourcing.html).

## Summary

I think it's really amazing that Elixir gives us tools to easily implement a mechanism to
restore the process state after it crashes. But be careful and don't just blindly use this
without thinking of the consequences. To be honest, I'm yet to see
a use case which will justify such behaviour. I think that it's much better to make sure
that the state is well protected instead of trying to recreate it after the failure
(now go and read this amazing post about [error kernels](https://medium.com/@jlouis666/error-kernels-9ad991200abd)).
After all, it's not about just letting it crash, it's about letting it crash
in a predictable way to be able to make the error handling simpler.

## Did I miss something?

Of course, I would really like to hear about good use cases for restoring the process state.
There's no comment box yet, but if you have anything to add, just send me an email
at [maciej@mkaszubowski.pl](mailto:maciej@mkaszubowski.pl) or tweet me at [@mkaszubowski94](https://twitter.com/mkaszubowski94)
and I'll be happy to add your comment at the
bottom of this article.
