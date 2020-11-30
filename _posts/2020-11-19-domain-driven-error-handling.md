---
layout: post
title:  "Domain-driven error handling. Don't handle errors - prevent them."
date:   2020-11-18 11:30:00 +0100
tags:  development modular-design elixir domain-driven-design
reading_time: 7
description: Four techniques that improve error handling in your system. Use domain-driven approach to design the software with failures in mind.
---

Our software pretends that success and error are binary. `:ok` or `:error`, `2XX` or `4XX/5XX`, something or `nil`, `Some` or `None`, `0` or `1`. Nothing in between.

Yet the real world is messy. The actions we make in the real world do not fall into clear success/failure categories. An action can be successful even if there are errors along the way.

Realizing this difference drastically improves error handling in your system.

Errors are an intrinsic part of the business domain. They should be a part of the domain model. It has to be designed with the same care as the happy path.

Here are a few techniques to reduce the complexity of your error handling and make it more domain-driven.

## Use defaults when fetching data

It's common to return `nil` or an error tuple (`{:error, :not_found}`) from a function that fetches data:

```elixir
@spec fetch_settings(String.t()) :: {:ok, Settings.t()} | {:error, :not_found}
def fetch_settings(user_id) do
  case Repo.get_by(Settings, user_id: user_id) do
    %Settings{} = settings -> {:ok, settings}
    nil -> {:error, :not_found}
  end
end
```

A piece of code like that is so common that we often do it without even thinking about it.

But often you can remove that branch of code by using sensible default values:

```elixir
@spec fetch_settings(String.t()) :: Settings.t()
def fetch_settings(user_id) do
  case Repo.get_by(Settings, user_id: user_id) do
    %Settings{} = settings -> settings
    nil -> default_settings()
  end
end

defp default_settings() do
  %Settings{
    # ...
  }
end
```

This is better for a few reasons:

1. The interface is smaller, which makes it easier to use for the client code. With the second version, the client can always expect a `Settings` struct back, so it doesn't have to do any branching.
2. We don't have to ever worry about *creating* an initial `Settings` entry for every user, so the interface shrinks.

## Use declarative interfaces

Using declarative and idempotent operations is a great way to reduce the complexity and decrease the amount of error handling (and edge cases in general).

Instead of telling the system what to do, describe the desired end state. Since the function is idempotent, we don't have to handle two possible scenarios on the client side:

```elixir
# Imperative interface, client has to interpret the error
@spec cancel(String.t(), String.t()) :: :ok | {:error, :aready_canceled}
def cancel(user_id, job_id) do
  case get_job(user_id, job_id) do
    %Job{status: "canceled"} ->
      {:error, :already_canceled}

    %Job{} = job ->
      do_cancel(job)
      :ok
  end
end

# Declarative interface, a single path to handle for the client
@spec cancel(String.t(), String.t()) :: :ok
def cancel(user_id, job_id) do
  case get_job(user_id, job_id) do
    %Job{status: "canceled"} ->
      :ok

    %Job{} = job ->
      do_cancel(job)
      :ok
  end
end
```

Some other examples:

- paying for order that was already paid for ("this function executed a payment" vs "an order should be paid after this function is called")
- deleting a deleted entry ("this function deletes X" vs "X shouldn't exist after this function is called")
- creating unique entries

In general - any case when the state of something is final (or not expected to be changed soon) can benefit from that.

## Don't fight real-world events

Data consistency is a big deal in software. But the real world cannot guarantee it.

People make mistakes, things go missing, people forget to put things into the software. The state of your system will not always reflect what's happening in the real world.

Is your system (or a specific part of it) a source of truth, or just a record of what's happened in the real world?

Consider the example code from the warehousing domain:

```elixir
def take_product_off_shelf(product_id, shelf_id) do
  if product_on_the_shelf?(product_id, shelf_id) do
    remove_from_shelf!(product_id, shelf_id)
  else
    {:error, :product_missing}
  end
end
```

When the employee is in the warehouse and uses our system to scan a product they're just taking off the shelf, returning such an error makes no sense.

When the request comes to the server - is it a command or an event? It's common to think that everything that comes from the user is a command - but it's false. The above scenario is a great example of a user-generated event.

In that case, the error is harmful for two reasons:

1. It confuses the user and gives them no way to react.
2. It makes us lose important information since the error will not prevent them from taking the item off the shelf.

We can make the system more useful by notifying the operators that something went wrong, rather than trying to prevent it:

```elixir
def take_off_shelf(product_id, shelf_id) do
  if product_on_the_shelf?(product_id, shelf_id) do
    remove_from_shelf!(product_id, shelf_id)
  else
    notify_operators_about_invalid_state(product_id, shelf_id)
  end
end
```

## Isolate clients from failing side-effect

Some business actions are side-effect-heavy:

```elixir
# After delivering the shipment, the driver should
# upload a photo of a confirmation as a delivery proof.
#
# This triggers a bunch of side-effects important for
# the entire process to move forward.

def upload_delivery_confirmation(user_id, shipment_id, params) do
  :ok = save_confirmation_photo(shipment_id, params)
  :ok = complete_shipment(shipment_id)
  :ok = send_notification_to_customer(shipment_id)
end

defp complete_shipment(user_id, shipment_id) do
  # Things that should happen:
  #
  # - Update statistics for the driver
  # - Update tracking information
  # - Pay the driver, send invoices
  # - Update delivery documentation
  # - Stop tracking the driver
end
```

Things can go wrong for many different reasons. An external system might be down, a database might time out, there might be a bug in some part of the code. Any of those things can make the entire function crash.

[Complex systems always run in downgraded mode](https://how.complexsystems.fail/#5). Any part of our system might fail at any time. Even if everything is fine most of the time, we still have to handle the case when it's not.

Why not design the system to treat failure as something normal? Thinking about failure scenarios is crucial for resilient systems.

It might make sense to assume success just after the photo is saved and notify the user about it. Once we have the photo safely stored, we can recover if something is wrong.

```elixir
def upload_delivery_confirmation(user_id, shipment_id, params) do
  :ok = save_confirmation_photo(shipment_id, params)

  # The rest of the side-effects are processed asynchronously using
  # at-least-once processing mechanism.
end
```

This means that the meaning of a successful response changes. It's no longer `we've successfully processed your request`, but `we'll process your request soon`. That's why it has to be a system-level change (also one of the reasons why UI/UX designers, developers, architects have to work closely with each other).

## Always think about the bigger context

Error handling is not limited to a single function. **Resilience is an emergent property of the entire system.**

Before you return an error consider a few things:

- Does the client code (or the end user) expect that error?
- Does the error have clear meaning and can be interpreted by the client code (or the end user)? Is the error part of the domain language or is it purely technical?
- Is the error useful? Does the client code (or the end user) know what to do to solve the problem? *Can* it solve the problem?
- In what circumstances can the error happen? A bug in the code, malicious behavior, user mistake, infrastructure problems, incorrect application state, human error outside the software? Each scenario requires a different strategy.

## There's no clear boundary between happy path and errors

Avoid that clear distinction between "happy path" and "error handling". Reality is messy. There's rarely a single happy path or a scenario that's clearly incorrect.

Error handling is not a technical problem. It's a business problem. By thinking about the meaning of errors we are forced to think about the broader context of our system. We start to notice the human aspect, the goals and expectations of the user.

Face the real world's complexity and messiness with its full richness, instead of trying to force every scenario into a binary correct/incorrect category.

## References

- [A Philosophy of Software Design](https://www.goodreads.com/book/show/39996759-a-philosophy-of-software-design) is where the original inspiration comes from. John Ousterhout encourages us to "define errors out of existence" to reduce the complexity of the software. I think that's a pretty nice choice of words. "Define" suggests it's a language problem. And since language is pretty important in Domain Driven Design, the connection is really interesting.
- Some of the concepts from this article come from my talk called [Error-free Elixir](https://www.youtube.com/watch?v=PwfOARkogDI) given at Code Elixir LDN 2019.
- The idea of declarative interfaces is heavily inspired by the Promise Theory. Learning how promises increase confidence was a huge help for me. If you work with cooperative systems - both computer and human systems - [Thinking in Promises](https://www.goodreads.com/book/show/24216682-thinking-in-promises) by Mark Burgess is an amazing read.
- [Designing Events-First Microservices](https://www.youtube.com/watch?v=1hwuWmMNT4c) talk by Jonas Bonér highlights the importance of embracing the real world and its complexity. It also mentions Promise Theory and its role in software design (not only for microservices).
