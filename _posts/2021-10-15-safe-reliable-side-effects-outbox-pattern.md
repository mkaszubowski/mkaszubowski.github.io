---
layout: post
title: "Safe and reliable side effects using the outbox pattern"
image: /assets/img/outbox-pattern/outbox-pattern.png
tags: patterns
reading_time: 3
description: The outbox pattern allows you to safely combine ACID transactions with external side effects.
canonical_url: https://blog.appsignal.com/2021/07/13/building-aggregates-in-elixir-and-postgresql.html
excerpt: |
    The outbox pattern allows you to safely combine ACID transactions with
    external side effects. This article describes how it works.
---

Database transactions are a great way to ensure data consistency for business transactions.

The problem is that not everything can be contained within the same database. Examples include:

- making requests to external APIs,
- calling different services,
- saving data to a different to database,
- operations that take long time,
- operations that await some input, e.g. human action.

How can we make sure those kinds of operations are consistent with the rest of the data?

Let's look at an example:

```elixir
defmodule Booking do
  def checkout(user_id, reservation_id) do
    reservation = get(user_id, reservation_id)

    Repo.transaction(fn ->
      :ok = mark_room_as_unavailable(reservation)
      :ok = mark_reservation_as_booked(reservation)
      :ok = send_confirmation(reservation)
    end)
  end
end
```

On the one hand, the transaction keeps the data consistent. If something fails, the previous steps are rolled back, leaving the system in a consistent state.

There are problems, though. **The confirmation (e.g., an email message) cannot be contained within the transaction.** That means that if the transaction fails, it cannot be rolled back. The confirmation can be sent even if the room hasn't actually been booked. This is bad!

The alternative is not perfect either:

```elixir
defmodule Booking do
  def checkout(user_id, reservation_id) do
    reservation = get(user_id, reservation_id)

    Repo.transaction(fn ->
      :ok = mark_room_as_unavailable(reservation)
      :ok = mark_reservation_as_booked(reservation)
    end)

    :ok = send_confirmation(reservation)
  end
end
```

In this scenario, there's no guarantee that the confirmation will actually be sent. If the function fails, or the server shuts down or is restarted, we'll miss the confirmation. Depending on your context and use case, this may or may not be acceptable (this pattern might be used for multiple different side effects, including sending notifications, charging money, sending domain events to different contexts/services etc.)

## The outbox pattern

If you need to ensure the side effect is executed, you can solve that using the outbox pattern. The outbox pattern works as follows:

![the outbox pattern](/assets/img/outbox-pattern/outbox-pattern.png)

1. You start a DB transaction.
2. You save any changes to the business entities (room and reservation).
3. You save the upcoming side effects ("send the confirmation") to the outbox.
4. You commit a transaction.
5. After the transaction, you pick up an unprocessed message from the mailbox and process it.
6. You remove each processed message from the outbox.

**The outbox is a table that lives in the same database as your business entities, so you can use a transaction to guarantee consistency of your data and side effects.** If other operations succeed, the message is guaranteed to be in the outbox. If the transaction is rolled back, the message will be rolled back, and side effect is not executed.

Using a separate background job to poll the messages from the mailbox guarantees that each message will be picked up even if the server shuts down just after committing the transaction. It also means that we don't have to wait for the job to finish to return to the caller.

## Why use the outbox pattern?

It's a simple, easy to implement, and reliable pattern that works for many use cases. It doesn't  require any complicated tools to start with (but can be easily integrated with them!). You only need:

- a simple DB table and a way to save all the parameters required by a side-effect
- some kind of periodic background job that picks the messages from the outbox

Those are things that are probably already used by your system. No need to add any other tools or infrastructure. In my book, it's a big win.

## Easy DB replication

A nice thing about the outbox pattern is that you can **switch your background jobs to use a DB replica without any risks or problems**. Since the data necessary by the background job is saved in the same transaction, as soon as the message appears in the replica's outbox, all the data required by the side effect processing code is already there!

Data consistency is a common problem when running background jobs from the DB replica. The outbox pattern solves those problems.

## Integrating with background jobs

Once you know the principles behind the outbox pattern, you can utilize it with any background jobs tools (although some, like [Oban](https://github.com/sorentwo/oban), already use your primary DB to save jobs, giving you the same benefits and guarantees.)

Even if the background job (and its arguments) is not saved in the same database (so it cannot be saved atomically with the rest of the data), you can use the outbox to let the background job know it's OK to continue.

```elixir
def checkout(user_id, cart_id) do
  Repo.transaction(fn ->
    reservation = get(user_id, reservation_id)

    :ok = mark_room_as_unavailable(reservation)
    :ok = mark_reservation_as_booked(reservation)

    outbox_message_id = generate_uuid()
    :ok = save_id_to_outbox(outbox_message_id)

    BackgroundJob.run(SendConfirmation, %{
      outbox_message_id: outbox_message_id
    })
  end)
end
```

When running the actual `SendConfirmation` code, the first thing you do is read the `outbox_message_id` from the outbox table.

If that id is there, this means that the transaction has been committed, and you can proceed. If it's not there, it indicates the transaction has not been committed. In that case, you can wait a bit or fail and retry. Setting a maximum number of retries (or maximum waiting period) will handle all the orphaned jobs. Note that this approach also works when running your background job with the DB replicas.

## References

- [https://microservices.io/patterns/data/transactional-outbox.html](https://microservices.io/patterns/data/transactional-outbox.html)
