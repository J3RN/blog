---
title: "A Gentle Introduction to Elixir Macros"
layout: post
date: 2023-11-05
description: |
  Macros are probably the most mystifying aspect of the Elixir programming language.  This post will attempt to explain them to programmers who have little experience with metaprogramming.
tags:
- elixir
- macros
aliases:
- /posts/2023-10-29-a-gentle-introduction-to-elixir-macros
draft: true
---

Macros are probably the most mystifying aspect of the Elixir programming language.  However, if you've written Elixir, you've almost certainly used macros.  Basic components of the language are implemented as macros, such as `if`, `&&` and `to_string`.  Popular Elixir libraries such as Ecto and Absinthe also make heavy use of macros to provide Domain-Specific Languages (DSLs) to simplify writing code for a specific domain (database queries and GraphQL schemas, here, respectively).

However, macros are a bit confusing because change the way that we normally think about how Elixir works.  Namely, the arguments to a macro aren't evaluated before the call.  That sounds a bit abstract, so let's look at an example.  Let's say we write a function `foo`:
```elixir
def foo(arg) do
  IO.inspect(arg)
end
```

If we invoke this function as `foo(1 + 1)`, `2` gets written to the console.  That's because `1 + 1` is evaluated *before* it gets to `foo`, and the value of the variable `arg` is `2`.

Let's make `foo` a macro instead:
```elixir
defmacro foo(arg) do
  IO.inspect(arg)
end
```

If we invoke this macro as `foo(1 + 1)`, then `{:+, [line: 7], [1, 1]}` is written to the console.  This may look like total gibberish to you, and that's OK!  We're going to get to how to read this next, but what it's trying to convey is that value of the variable `arg` represents the expression `1 + 1`, **not** `2`.

OK, let's talk about that tuple.  The tuple is Elixir's way of representing code as data.  If you were tasked to represent code as data—to store it in a database, say—you might think to use a string to represent that code, like so:
```elixir
"1 + 1"
```

This is fine for some purposes, but there are a couple issues with this approach.  The first is that there's no guarantee that the code in the string is actually valid.  For instance, `"foo("` isn't valid Elixir code, but it is a valid string.  Secondarily, the string doesn't provide any easy ways to understand the _structure_ of the code, which may be important for your use-case.  For instance, it would be nice if the datastructure representing the code conveyed that the code consists of an operator being invoked with two arguments.  The "quoted form" of Elixir code addresses these shortcomings.

The code
```elixir
1 + 1
```

has the quoted form:
```elixir
{:+, [context: Elixir, imports: [{1, Kernel}, {2, Kernel}]], [1, 1]}
```

This, admittedly, looks pretty scary!  However, if we ignore the second element of this tree-element tuple, its intent starts to come into focus:
```elixir
{:+, _, [1, 1]}
```

This tuple is trying to convey that the operator `+` being invoked with two arguments, `1` and `1`.  The lengthy second element of the tuple is metadata: Which modules are imported, what line the code appeared on, etc.  This metadata is useful for a variety of purposes, but makes the quoted form a bit messy to look at.

Let's look at a second example.  The code
```elixir
to_string(:hello)
```

has the quoted form
```elixir
{:to_string, [context: Elixir, imports: [{1, Kernel}]], [:hello]}
```

Ignoring the second element again, we have:
```elixir
{:to_string, _, [:hello]}
```

The function `to_string` here is being invoked with a single atom, `:hello`.  See it?

To get the quoted form for any code, you can use the `quote` macro like so:
```elixir
iex(1)> quote do
...(1)>   to_string(:hello)
...(1)> end
{:to_string, [context: Elixir, imports: [{1, Kernel}]], [:hello]}
```

Please take some time to play with the `quote` macro until you really get the hang of it.

As we saw above, arguments to macros are not evaluated and, instead, their quoted form is given to the macro.  This can be used to perform some interesting tricks!  For instance, using what we know about Elixir's quoted form, we could write a macro that tells us whether the argument represents an addition or not:

```elixir
defmodule MacroTest do
  defmacro addition?({:+, _, _}), do: true
  defmacro addition?(_), do: false
end
```

When using this in IEx, we'll first need to require `MacroTest`, then we can use our `addition` macro:
```elixir
iex> require MacroTest
MacroTest
iex> MacroTest.addition?(1 + 1)
true
iex> MacroTest.addition?(5 + 7)
true
iex> MacroTest.addition?(to_string(:hello))
false
```

Nifty!  There's also something important to note here:
```elixir
iex> MacroTest.addition?(IO.puts("Hello!"))
false
```

Notice how "Hello!" wasn't printed to the console?  That code never ran.  Elixir converted it to its quoted form, `MacroTest.addition?` checked to see if it was addition, and then the code was gone; it left scope without ever being invoked.  This is a useful feature, because it implies a power of macros: The code that you pass them doesn't have to be valid.  For instance, if you tried to add an atom to an integer, normally it would raise an error:

```elixir
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression: :foo + 1
    :erlang.+(:foo, 1)
    iex:28: (file)
```

However, because `MacroTest.addition?` doesn't ever try to run this code, the fact that it's invalid is irrelevant:
```elixir
iex> MacroTest.addition?(:foo + 1)
true
```

Once you internalize this fact, you can begin to realize how Ecto is able to convert `not is_nil` to `IS NOT NULL` and other such tricks.

Speaking of which, we need to discuss the other important feature of macros: Not only do are their *arguments* quoted forms, their *return value* is also a quoted form.  With `MacroTest.addition?`, we were getting by on a technicality: `true` and `false` are the quoted forms of `true` and `false`.

```elixir
iex> quote do: true
true
iex> quote do: false
false
```

Most Elixir primitives are their own quoted forms, in fact.  But enough about that, let's write a macro that returns a more interesting quoted form!

```elixir
defmodule MacroTest2 do
  defmacro two() do
    {:+, [], [1, 1]}
  end
end
```

The quoted form being returned from the `MacroTest2.two` macro you should recognize as the quoted form of `1 + 1` with the metadata replaced by an empty list.  Empty metadata is likely to confuse static analysis tools (e.g. Dialyzer), but that's not important for this example.  Let's try it!

```elixir
iex(34)> require MacroTest2
MacroTest2
iex(35)> MacroTest2.two()
2
```

`MacroTest2.two()` returns the quoted form of `1 + 1` which is then executed as code by IEx to yield `2`.  In a way, we've come full circle from where we started!

"But surely," you say, "macro authors aren't writing out these quoted forms by hand."  Indeed, they generally don't!  Our good friend `quote`, mentioned before, converts code to it's quoted form, and so `MacroTest2.two` could be rewritten to use `quote` like so:

```elixir
defmodule MacroTest2 do
  defmacro two() do
    quote do
	  1 + 1
	end
  end
end
```

Indeed, this is how many macros are written: their return value is the result of a `quote` invocation.
