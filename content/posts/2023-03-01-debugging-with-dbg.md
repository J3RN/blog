---
layout: post
title: "Debugging Elixir with Erlang's dbg"
date: 2023-03-01
description: If you've ever wanted to see what's going on in your codebase without inserting `IO.inspect` everywhere, Erlang's `dbg` has a solution for you!
tags:
- elixir
- erlang
---

Sometimes, either in local development or on a production server, you want to see when a given function is called, what arguments it is being called with, and what it's returning.  Let's say that you have a `MyApp.Widgets.show_widget?/1` function which consumes a widget and returns a boolean indicating whether or not that widget should be shown to the user.  However, the logic contained in there is pretty complicated and&mdash;worse&mdash;you don't know really where it is called from anyway.

Enter Erlang's `dbg`, a module that uses Erlang's powerful tracing system to print events of interest to the console.  Its usage in this article works regardless of whether you're using an IEx prompt or injecting this code into a test.

To use `dbg` to solve our mysteries, we'll first have to start its tracer process[^1]:

```elixir
:dbg.tracer()
```

Next, we'll need to tell `dbg` what to look for, which we do with the unusually terse `p` function (all function names in `dbg` are, for some reason, very terse):

```elixir
:dbg.p(:all, :call)
```

The function name `p` is supposedly short for "**p**rocess", and its purpose is to tell `dbg` which tracer events we're interested in.  The first argument tells `dbg` which process(es) you want to trace.  You can pass a PID here to trace a specific process, but in the example we're passing the atom `:all` to indicate that we want events from _all_ processes.  There are other special atoms you can use to indicate groups of processes, such as `:new_processes`; check [the `:dbg.p/2` documentation] for a full list.  The second argument specifies which kinds of tracer events you want to be notified about. `:call` indicates that we want to know about function calls, and there are other options such as `:send` for message sending, `:ports` for things related to ports, etc.

Well, `dbg` refuses to print information about _every_ function call being made in the system&mdash;at least, unless you ask it to&mdash;so in addition to telling `dbg` to trace function calls across all processes, we'll need to specify which function calls it should trace.  We'll do that here with the `tpl` function:

```elixir
:dbg.tpl(MyApp.Widgets, :show_widget?, :x)
```

The function name `tpl` is short for "**t**race **p**attern **l**ocal."  The first two arguments indicate, as you probably guessed, the module and function that you want to trace.  That mysterious `:x`?  We'll get to that in a moment.

The `dbg` docs will generally point you towards using `tp`.  The difference between `tp` and `tpl` is that `tp` only traces "global" calls&mdash;calls between modules&mdash;and `tpl` traces both global and "local" (intra-module) calls.  I'm generally a "more data is better" person, so I prefer `tpl`.

That's all the setup we need to start debugging!  After this code has been evaluated in a running system, events that match our "pattern" (here, calls of the function `MyApp.Widgets.show_widget/1` in any process) will be printed to the console.  Here's an example:


```elixir
iex(4)> MyApp.Widgets.show_widget?(%MyApp.Widget{spurving_bearings: false})
# (<0.187.0>) call 'Elixir.MyApp.Widgets':'show_widget?'(#{'__struct__' => 'Elixir.MyApp.Widget',hydrocoptic_marzelvanes => nil,
#   spurving_bearings => false})
# (<0.187.0>) returned from 'Elixir.MyApp.Widgets':'show_widget?'/1 -> false
false
```

As you can see above, `dbg` prints out that our function was called, what arguments it was called with, and what the function returned.  This is not super exciting since we called it manually, but when you're at least one step removed...


```elixir
iex(5)> MyApp.list_widgets()
# (<0.187.0>) call 'Elixir.MyApp.Widgets':'show_widget?'(#{'__struct__' => 'Elixir.MyApp.Widget',
#   hydrocoptic_marzelvanes => [a,b,c,d,e,f],
#   spurving_bearings => true})
# (<0.187.0>) returned from 'Elixir.MyApp.Widgets':'show_widget?'/1 -> true
[
  %MyApp.Widget{
    spurving_bearings: true,
    hydrocoptic_marzelvanes: [:a, :b, :c, :d, :e, :f]
  }
]

```

we can see that `MyApp.Widgets.show_widget?/1` was called, what it was passed, and what it returned.

OK, let's talk about `:x`.  The third argument to `tpl` is a "Match Spec".  Match specifications are non-trivial, and if you want to learn more about them you can do that here: [Match Specifications in Erlang].  Luckily for us, `tpl` has a few "built-in aliases" for debugging-related match specs, of which `:x` is one.  Here, `:x` is short for "e**x**ceptions," which is also a bit deceiving since, in addition to reporting exceptions, it also reports calls, arguments, and return values.  The other two built-in aliases are `:c` and `:cx`.  `:c` is short for "**c**aller" and it reports the function that called your function of interest:

```elixir
iex(7)> MyApp.list_widgets()
# (<0.187.0>) call 'Elixir.MyApp.Widgets':'show_widget?'(#{'__struct__' => 'Elixir.MyApp.Widget',
#   hydrocoptic_marzelvanes => [a,b,c,d,e,f],
#   spurving_bearings => true}) ({'Elixir.Enum',filter_list,2})
[
  %MyApp.Widget{
    spurving_bearings: true,
    hydrocoptic_marzelvanes: [:a, :b, :c, :d, :e, :f]
  }
]
```

At the very end of the output you can see that `({'Elixir.Enum',filter_list,2})` is the function that called `MyApp.Widgets.show_widget?/1`.  That nice to know, but you may have noticed that `:c` doesn't report the return value.  If you want both the caller and the return value, `:cx` is for you:

```elixir
iex(9)> MyApp.list_widgets()
# (<0.187.0>) call 'Elixir.MyApp.Widgets':'show_widget?'(#{'__struct__' => 'Elixir.MyApp.Widget',
#   hydrocoptic_marzelvanes => [a,b,c,d,e,f],
#   spurving_bearings => true}) ({'Elixir.Enum',filter_list,2})
# (<0.187.0>) returned from 'Elixir.MyApp.Widgets':'show_widget?'/1 -> true
[
  %MyApp.Widget{
    spurving_bearings: true,
    hydrocoptic_marzelvanes: [:a, :b, :c, :d, :e, :f]
  }
]
```

One final note before you go: You may have noticed that the reports from `dbg` use Erlang syntax, and that's for the very good reason that `dbg` is an Erlang utility and knows nothing about Elixir and its syntax.  Printing to the console is performed by a "handler" function which is an optional argument to `dbg:tracer`.  By default the handler function is `dbg:dhandler/2`, but there's no reason why an Elixir-formatting handler couldn't be written.  That said, [`dbg:dhandler/2` (and it's closely associated `dhandler1/3`) constitute nearly 200 lines of code](https://github.com/erlang/otp/blob/2e9f3f57dc6b3c7e4ce48a9955abf16e3dc6c16d/lib/runtime_tools/src/dbg.erl#L999-L1162), so it's no small task!

I hope you found this information useful and happy debugging!

[^1]: The process spawned by `:dbg.tracer()` (aptly named `dbg`) listens for events from Erlang's tracing system.

[the `:dbg.p/2` documentation]: https://www.erlang.org/doc/man/dbg.html#p-2
[Match Specifications in Erlang]: https://www.erlang.org/doc/apps/erts/match_spec.html
