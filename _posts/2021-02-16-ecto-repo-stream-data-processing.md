---
layout: post
title: "How to use Ecto's Repo.stream/1 to process large amounts of data"
date: 2021-02-16 11:30:00 +0100
tags: elixir development
reading_time: 2
description: How to process large amounts of data at once using Ecto and Repo.stream/1
excerpt: |
  This short post shows a really easy way to process large-ish (max few hours of processing)
  datasets using Ecto's Repo.stream/1.
---

{: .tip}
**This is not a post about Big Data.** This is about datasets that are
too big to handle entirely in-memory but too small to justify spending a few
hours writing a custom data pipeline.

Here is an easy way to deal with data that should take up to a few hours to
process:

```elixir
Repo.transaction(fn ->
  YourSchema
  |> order_by(asc: :inserted_at)
  |> any_query()
  |> Repo.stream()
  |> Stream.map(fn user -> any_transformation(user) end)
  |> Stream.filter(&any_filter/1)
  |> Stream.each(fn user -> do_something_with_user(user) end)
  |> Stream.run()
end, timeout: :infinity)
```

## Comments

- `Repo.stream/1` has to be used inside a transaction for both PostgreSQL and
  MySQL. It queries the DB in chunks (the default chunk size is 500 rows.)
- We're passing `timeout: :infinity` to `Repo.transaction/2` to allow it to run
  as long as it needs. [The default is 15
  seconds](https://hexdocs.pm/ecto/Ecto.Repo.html#module-shared-options), after
  which the transaction times out.
- Streaming the results will block one of the connections from Ecto's connection pool, so watch out for that if your connection pool size is small.
- Since it's run in a transaction, rows added after the function starts won't be
  read. You can re-run the script adding a `where` clause with a timestamp to
  synchronise those entries.
- Since `Repo.stream` returns an Elixir stream, you have to use `Stream` module instead of `Enum`.
  You can use `Stream` instead of `Enum` for most of what you'd typically need.
  Just use [Stream.run/1](https://hexdocs.pm/elixir/Stream.html#run/1) at the end to actually run the
  processing.
- Before you run the function on the entire dataset, you can easily add `|>
  limit(10)` clause to your query and test if processing is correct.

## Example use cases

- Migrate the data from one table to another.
- Migrate the data to another database (for example: build an initial index in
  ElasticSearch).
- Export the data to a CSV file.
- Fill missing values for old data (after adding a new column).
- Any manual, one-time task you might want to do for some data.

## Alternative approaches

- If you only need CSV and don't need to do any in-memory transformations, [you
  can export data from PostgreSQL
  directly](https://www.postgresqltutorial.com/export-postgresql-table-to-csv-file/)
  using the `COPY` statement. You can import data from CSV, too.
- If you want to make the processing parallel, take a look at
  [Flow](https://github.com/dashbitco/flow) (or
  [GenStage](https://github.com/elixir-lang/gen_stage))

## References:

- [https://hexdocs.pm/ecto/Ecto.Repo.html#c:stream/2](https://hexdocs.pm/ecto/Ecto.Repo.html#c:stream/2)
- [https://hexdocs.pm/elixir/Stream.html](https://hexdocs.pm/elixir/Stream.html)
