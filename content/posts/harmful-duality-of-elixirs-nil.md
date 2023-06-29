---
layout: post
title: "The Harmful Duality of Elixir's nil"
date: 2020-12-13 20:46:00 -0500
description: |
  In Elixir, nil can mean "what was sought was not found" and also "this is intentionally blank". This disparity in the intentionality of nil makes life difficult for Elixir developers.
tags:
- elixir
- nil
- functional programming
aliases:
- /posts/2020-12-13-harmful-duality-of-elixirs-nil
---

### The Problem

Have you ever written code like this?

```elixir
def get_address(address_id) do
  case Addresses.get(address_id) do
    nil -> {:error, :address_not_found}
    val -> {:ok, val}
  end
end
```

I have. This pattern is commonly used to make the value that [Ecto.Repo.get/3](https://hexdocs.pm/ecto/Ecto.Repo.html#c:get/3) returns (a struct or `nil`) play well with Elixir's [`with`](https://hexdocs.pm/elixir/Kernel.SpecialForms.html#with/1) construct. Since `Ecto.Repo.get/3` will always return some single value, a `nil` check is required to determine whether that value indicates a success or a failure. Doing this check inline is somewhat clumsy:

```elixir
with address when not is_nil(address) <- Addresses.get(address_id) do
```

There's a lot happening in that line, which makes the alternative attractive:

```elixir
with {:ok, address} <- get_address(address_id) do
```

This is a minor complaint, but it points to a much larger issue with `nil` in Elixir. First, consider the fact that `nil` serves two purposes:
1. Indicate when something cannot be found
2. Represent an empty value; nothingness

While Elixir gives us the convention of `:error` as the "operation did not succeed" value, in specific cases `nil` will also serve that purpose. Why is this an issue? Let me show you an example.

```elixir
iex(1)> map1 = %{key: nil}
%{key: nil}
iex(2)> map2 = %{}
%{}
iex(3)> Map.get(map1, :key)
nil
iex(4)> Map.get(map2, :key)
nil
```

In this example, [`Map.get/3`](https://hexdocs.pm/elixir/Map.html#get/3) returns `nil` when passed `map1` or `map2`, but for _different reasons_. In a less contrived scenario, this can lead to some head-scratching bugs. Consider the case where the value associated with a key is intentionally `nil` to clear out some previously set value in a record. If you were using `Map.get/3` to lookup the key in the map, the return value would not indicate if the key and value simply weren't supplied (and thus no corresponding update to the record should occur) or if they were supplied and the value happened to be `nil` (and the old value in the record should be cleared).

This issue is also present with [`Keyword.get/3`](https://hexdocs.pm/elixir/Keyword.html#get/3), [`Access.get/3`](https://hexdocs.pm/elixir/Access.html#get/3), and even [`Application.get_env/3`](https://hexdocs.pm/elixir/Application.html#get_env/3).

### The Remedy

As I alluded to earlier, the remedy here is to take away one of `nil`'s jobs, specifically in indicating failure. The astute among you may have already realized that the third argument (not used in my examples above) to the various `get/3` functions is a "default". You could certainly change the default to `:error` whenever you call these functions, but there's likely an even better option.

**Enter `fetch/2`**. For each `get/3` function I previously mentioned, there is a `fetch/2` equivalent (specifically [`Map.fetch/2`](https://hexdocs.pm/elixir/Map.html#fetch/2), [`Keyword.fetch/2`](https://hexdocs.pm/elixir/Keyword.html#fetch/2), [`Access.fetch/2`](https://hexdocs.pm/elixir/Access.html#fetch/2), and [`Application.fetch_env/2`](https://hexdocs.pm/elixir/Application.html#fetch_env/2)). These functions work similarly to their `get/3` counterparts, but return `{:ok, value}` when they succeed and `:error` otherwise. Consider the `Map.get/3` example from earlier, but using `Map.fetch/2` instead:

```elixir
iex(1)> map1 = %{key: nil}
%{key: nil}
iex(2)> map2 = %{}
%{}
iex(3)> Map.fetch(map1, :key)
{:ok, nil}
iex(4)> Map.fetch(map2, :key)
:error
```

There is now a clear distinction between the case where the key and value exist and is the value happens to be `nil` and the case where the lookup has failed. The added bonus of `fetch/2` over specifying a default to `get/3` is that `fetch`'s `{:ok, value}` return is more `with`-friendly.

```elixir
with {:ok, value} <- Map.fetch(map1, :key) do
  perform_operation(value)
else
  :error -> {:error, "Value not found"}
end
```

"But wait," you might say, "what about the example with `Ecto.Repo.get/3`? Is there an `Ecto.Repo.fetch/2`?"

Sadly, the answer is "no." Your only options when looking up a record by its primary key are `Ecto.Repo.get/3` and [`Ecto.Repo.get!/3`](https://hexdocs.pm/ecto/Ecto.Repo.html#c:get!/3) (which raises an error when the record is not found).

In a way, this makes sense. Since `Ecto.Repo.get/3` will always return a struct on success, there is no way a `nil` return could be confused for anything less than a lookup failure. However, by choosing `nil` as the failed lookup value, Ecto has pigeon-holed `nil` into its usage that makes less sense. Should a `nil` returned by `Ecto.Repo.get/3` propagate out the context of Ecto, the confusion over whether it constitutes a failure or an intentionally blank value will ensue.

While the ship has probably sailed on Ecto's API, it hasn't on other libraries and your own code. Whenever you type those three characters into your text editor, make sure it means precisely one thing.
