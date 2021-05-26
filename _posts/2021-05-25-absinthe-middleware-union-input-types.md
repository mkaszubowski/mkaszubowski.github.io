---
layout: post
title: "Absinthe middleware for tagged union input types"
date: 2021-05-25 11:30:00 +0100
tags: elixir graphql
reading_time: 5
image: /assets/img/union-type.png
description: An example Absinthe middleware that validates union input types for GraphQL.
excerpt: |
  While GraphQL and Absinthe don't support union input types yet, you can simulate them
  by using a middleware to add automatic validations to your schema.
---

*At the time of writing this, union input types are not yet supported by either
[Absinthe](https://github.com/absinthe-graphql/absinthe) or the GraphQL spec
itself. [This RFC in the
spec](https://github.com/graphql/graphql-spec/blob/main/rfcs/InputUnion.md)
describes possible solutions and their challenges.*

*The leading solution seems to be a tagged union. In this post, I'll describe my
attempt of implementing that as an Absinthe middleware.*

## What's a union type

A [union type](https://hexdocs.pm/absinthe/Absinthe.Type.Union.html) is one of
GraphQL's Abstract Data Types that's made up of multiple possible types.

Using the example domain from the [RFC
document](https://github.com/graphql/graphql-spec/blob/main/rfcs/InputUnion.md),
let's say that we model an animal shelter and we have two different animal
species we support: cats and dogs.

An union type allows us to model each animal as one of those two different
types:

```elixir
object :shelter do
  field :location, non_null(:string)
  field :animals, list_of(non_null(:animal))
end

union :animal do
  types [:cat, :dog]

  resolve_type fn
    %{name: _, lives_left: _} -> :cat
    %{name: _} -> :dog
  end
end

object :dog do
  field :name, non_null(:string)
end

object :cat do
  field :name, non_null(:string)
  field :lives_left, non_null(:string)
end
```

{: .tip}
Another Abstract Data Type (ADT) supported by GraphQL and Absinthe is
the [interface](https://hexdocs.pm/absinthe/Absinthe.Type.Interface.html), which
might have made more sense for querying this data, but let's ignore that for the
sake of consistency.

## Unions for input objects?

You can imagine that the same would be helpful in input objects and mutations.
If you want to log new animals at the shelter, how do you distinguish between
cats and dogs?

A popular solution is to use a tagged unions for that:

```elixir
mutation do
  field :log_animal_drop_off, :shelter do
    arg :location, non_null(:string)
    arg :animals, list_of(non_null(:animal_input))

    resolve &AnimalDropoff.resolve/2
  end
end

input_object :animal_input do
  field :cat, :cat_input
  field :dog, :dog_input
end

input_object :cat_input do
  field :name, non_null(:string)
  field :lives_left, non_null(:integer)
end

input_object :dog_input do
  field :name, non_null(:string)
end
```

This solution assumes that clients will pass only one of the fields in the
`animal_input` object. Unfortunately, there's no automatic validation for that,
except for the validations you might already have in the resolver or business
logic.

## The middleware

Before we get native support for union input types in GraphQL and Absinthe, you
can use the following
[middleware](https://hexdocs.pm/absinthe/Absinthe.Middleware.html) to add
automatic validations for tagged unions:

```elixir
defmodule MyGraph.Middleware.OneOfInputValidation do
  @moduledoc """
  Middleware that allows you to specify one-of/union input_objects.

  In a one-of/union input_object, only one of the keys can be set at any given
  time.

  This middleware must be called before `resolve` to prevent mutation from
  running if the validation fails.

  The `keys` argument specifies the path to the one-off input object, following
  a similar pattern to the one found in `Kernel.get_in/2`, but is extented to
  support lists.
  """

  @behaviour Absinthe.Middleware

  @impl true
  def call(resolution = %{state: :resolved, errors: []}, _keys) do
    IO.warn("#{__MODULE__} must be called before a field is resolved.")

    resolution
  end

  @impl true
  def call(resolution = %{state: :resolved}, _keys) do
    resolution
  end

  @impl true
  def call(resolution, key) when is_atom(key), do: call(resolution, [key])

  @impl true
  def call(%{state: :unresolved} = resolution, keys) when is_list(keys) do
    field_values =
      resolution.arguments
      |> get_values(keys)
      |> to_flat_list()

    if Enum.any?(field_values, &more_than_one_value?/1) do
      field_name =
        keys
        |> List.last()
        |> to_string()
        |> Absinthe.Utils.camelize(lower: true)

      error = "You can only set one of the values in: '#{field_name}'."
      resolution |> Absinthe.Resolution.put_result({:error, error})
    else
      resolution
    end
  end

  # Inspired by
  # https://github.com/elixir-lang/elixir/blob/v1.11.4/lib/elixir/lib/kernel.ex#L2473
  # but adjusted to handle lists
  defp get_values(nil, _), do: []

  defp get_values(data, []), do: data

  defp get_values(data, keys) when is_list(data),
    do: Enum.map(data, &get_values(&1, keys))

  defp get_values(map, [key | rest]) when is_map(map),
    do: get_values(map[key], rest)

  defp to_flat_list(list) when is_list(list),
    do: List.flatten(list)

  defp to_flat_list(element), do: [element]

  defp more_than_one_value?(map) when is_map(map),
    do: map |> Map.keys() |> length() > 1

  defp more_than_one_value?(_), do: false
end
```

And use it like this in your schema:

```elixir
mutation do
  field :log_animal_drop_off, :shelter do
    arg :location, non_null(:string)
    arg :animals, list_of(non_null(:animal_input))

    middleware MyGraph.Middleware.OneOfInputValidation, [:animals]

    resolve &AnimalDropoff.resolve/2
  end
end
```

Or, if union field is nested, you can specify the path to the field,
similar to [get_in/2](https://hexdocs.pm/elixir/Kernel.html#get_in/2):

```elixir
field :log_animal_drop_off, :shelter do
  arg :input, non_null(:animal_drop_off_input)

  middleware MyGraph.Middleware.OneOfInputValidation, [:input, :animals]

  resolve &AnimalDropoff.resolve/2
end
```

## Could you help me find a better solution?

I'm neither a GraphQL nor Absinthe expert, so this is far from perfect. It's a
first step, but I'm sure someone can (or already have) find a better solution.

If you are that someone and you have some ideas on how to improve it (or see
what's wrong), [reach out and let me know](https://twitter.com/mkaszubowski94)!
