---
layout: post
title: "Exciting Changes Coming in Elixir 1.12"
date: 2021-04-14 15:10:00 -0500
description: |
  I'm dedicating this post to trawling through the full Elixir 1.12 changelog and shining a spotlight on a few of my favorite changes.
showToc: true
tags:
- elixir
aliases:
- /posts/2021-04-14-elixir-1.12
---

José recently [announced the availability of Elixir 1.12.0 rc.0](https://github.com/elixir-lang/elixir/releases/tag/v1.12.0-rc.0), a precursor to the full Elixir 1.12.0 release which should come within the next month or so. This update consists primarily of developer quality-of-life improvements. In his announcement, José elaborated on a few changes, but I couldn't help but feel that some exciting changes were relegated to footnotes. As such, I'm dedicating this post to trawling through the full changelog and shining a spotlight on a few of my favorite changes.

### Mix.install

Let's say that there's a cool new Elixir library that you'd like to try out. In previous Elixir versions, experimenting with a new library would require you to create a new Mix project, add the library as a dependency, and start an IEx shell in the project (`iex -S mix`). Well, dabblers rejoice! [A recent PR by Wojtek Mach](https://github.com/elixir-lang/elixir/pull/10674) adds the ability for one to install an arbitrary package from Hex inside an IEx shell by running `Mix.install/1`.

```elixir
Interactive Elixir (1.12.0-rc.0) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> Mix.install([:finch])
:ok
iex(2)> Finch.start_link(name: MyFinch)
{:ok, #PID<0.223.0>}
```

In this example, I opened a new IEx shell outside of any project, and used `Mix.install` to pull in the [Finch HTTP library](https://github.com/keathley/finch) that there's been [much hubbub about lately](https://twitter.com/ChrisKeathley/status/1364692787032113153). This feature is so convenient, I have a feeling I'll be doing this a whole lot going forward!

### Integer.pow/2 and Float.pow/2

This may strike you as surprising, but Elixir has no `**` exponentiation operator and, until version 1.12, no `pow` function. Previously, developers who needed to use exponents were directed to use [Erlang's `math:pow/2`](https://erlang.org/doc/man/math.html#pow-2). Well, fret not, `pow`-er users!

```elixir
iex(1)> Integer.pow(2, 3)
8
```

The `Integer.pow/2` function requires both the base and the exponent to be integers. If you have a float that you need to raise to an exponent, `Float.pow/2` is the function for you.

```elixir
iex(1)> Float.pow(2.0, 0.5)
1.4142135623730951
```

Under-the-hood, `Integer.pow/2` uses an [exponentiation by squaring](https://en.wikipedia.org/wiki/Exponentiation_by_squaring) approach whereas `Float.pow/2` simply calls Erlang's `math:pow/2`. **Beware**, neither of these function supports raising an integer to a float power (e.g. `Float.pow(2, 0.5)`). For that edge-case, you'll either need to cast your integer base to a float or call Erlang's more lenient `math:pow/2`.

### Stepped Ranges

Elixir has had ranges for quite some time written in the form `first..last`. Elixir will determine whether the step between the numbers should be either `1` or `-1` by comparing the "first" and "last" numbers.

```elixir
iex(1)> Enum.to_list(1..10)
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

iex(2)> Enum.to_list(10..1) 
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]
```

José pointed out three limitation with ranges in [his proposal on the mailing list](https://groups.google.com/g/elixir-lang-core/c/U5EhplEqda4):
1. It is not possible to have ranges with custom steps
2. It is not possible to have empty ranges
3. Users may accidentally forget to check the range boundaries

His first concern is very clear and the last is referring to pattern-matching on ranges which I have never actually seen in production. However, I found his second listed limitation pretty intriguing, so I tried to create an empty range:

```elixir
iex(1)> Enum.to_list(1..1)
[1]

iex(2)> Enum.to_list(1..0)
[1, 0]
```

Sure enough, no dice.

The exact syntax for this feature was hotly debated on the mailing list, but was ultimately decided to be `first..last//step`, where `step` is optional if it is `1` or `-1`. An inferred step of `-1` will be deprecated in future versions, though.

```elixir
iex(1)> 1..10
1..10

iex(2)> 1..10//2
1..10//2

iex(3)> Enum.to_list(1..10//2)
[1, 3, 5, 7, 9]

iex(4)> 10..1
10..1//-1

iex(5)> Enum.to_list(10..1)
[10, 9, 8, 7, 6, 5, 4, 3, 2, 1]

iex(6)> Enum.to_list(1..0//1)
[]
```

As you can see from the last example, Elixir's ranges can now represent an empty range if the step does not agree with the direction from `first` to `last`. I imagine that this will provide an elegant solution to some annoying bugs.

Notably, this feature was also applied to dates as `Date.range/3` without new syntax. If you want every-other day in January 2021, you can now write the following:

```elixir
iex(1)> Date.range(~D[2020-01-01], ~D[2020-01-31], 2)
#DateRange<~D[2020-01-01], ~D[2020-01-31], 2>
```

### Kernel.then/2

This feature was relegated to a footnote (alongside `Integer.pow/2`), but I think it warrants better. I had initially assumed that `then/2` was for handling error types in a pipe, something like [Brex.Result](https://hexdocs.pm/brex_result/readme.html), but I was way off-base. `then/2` simply allows you to pipe a value into an anonymous function.

```elixir
iex(1)> 1 |> then(&(&1 * 2))
2
```

If you thought that piping values into anonymous functions was something that Elixir could already do, you'd be _technically_ correct. The syntax is pretty gnarly, though, since the invocation of an anonymous function is performed as `fun.()` (notice the `.`). Therefore, you had to wrap your anonymous function in an invocation:

```elixir
iex(1)> 1 |> (&(&1 * 2)).()
2
```

See those parentheses surrounding the anonymous function? Yeah, those are mandatory. If you're like me, you prefer the cleanliness of `then/2`.

### Kernel.tap/2

`tap/2` has a strong resemblance to `then/2`, but with a key difference: the value created by the anonymous function given to `tap/2` is discarded.

```elixir
iex(1)> 1 |> tap(&IO.puts(&1)) |> then(&(&1 * 2))
1
2
```

You may know that `IO.puts/1` returns `:ok`, but that fact largely irrelevant here since that `:ok` is discarded, and the original value given to `tap/2` (here, `1`) is passed out unchanged.

Just like [Ruby's `tap` method](https://ruby-doc.org/core-2.6.1/Object.html#method-i-tap), which was almost assuredly the inspiration for this feature, the value of `tap/2` lies in performing some IO with the value in the pipe chain. For instance, if you're interested at what the value being passed through a pipe chain is at a given point, you can simply inject `|> tap(&IO.puts(&1))` (as in the above example).

### Formatter will not add newlines around interpolation

This is very small, but has driven me crazy for a while now. Given a long string with interpolation, for example:

```elixir
"cannot build datetime with #{inspect(date)} and #{inspect(time)}, reason: #{inspect(reason)}"
```
_(this example comes from Elixir's [DateTime.new!/4](https://github.com/elixir-lang/elixir/blob/d7f0c87bc52e426aa5b77ec7a4334fa437daa5c5/lib/elixir/lib/calendar/datetime.ex#L244))_

The Elixir formatter in versions < 1.12 will format this line as:
```elixir
"cannot build datetime with #{inspect(date)} and #{inspect(time)}, reason: #{
  inspect(reason)
}"
```

I think this looks just terrible. The good news for me is that as of Elixir 1.12, the formatter will now leave that line as-is!

### IEx learns about pipes

At some point you've probably tried to copy-and-paste a pipe chain from your application into an IEx console and encountered this error:
```elixir
iex(1)> Post
Post
iex(2)> |> where([p], p.title == "Exciting Changes Coming in Elixir 1.12")
** (SyntaxError) iex:4:1: syntax error before: '|>'
```

Well, no longer! As of Elixir 1.12, IEx will now handle pipes!

```elixir
iex(1)> Post
Post
iex(2)> |> where([p], p.title == "Exciting Changes Coming in Elixir 1.12")
#Ecto.Query<from p0 in MyApp.Post, where: p0.title == "Exciting Changes Coming in Elixir 1.12">
iex(3)> |> Repo.all()
[]
```

### Conclusion

As you've probably deduced, there's a lot to love about Elixir 1.12. It's worth noting that while I covered quite a few things here, there is still far more that I didn't which you can find in [the announcement](https://github.com/elixir-lang/elixir/releases/tag/v1.12.0-rc.0). I don't know about you, but I'm quite excited for the full Elixir 1.12 release!
