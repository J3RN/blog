---
title: "Let it Crash?"
layout: post
date: 2024-01-05
description: 
draft: true
tags:
- elixir
- erlang
- software development
---

In the BEAM ecosystem ([Erlang], [Elixir], [Gleam], etc), there's a common idiom:

## Let It Crash

And your first reaction to this is probably astonishment, because crashing is, generally-speaking, not ideal.  In traditional single-threaded systems—in server settings—a crash means that your program dies and it's then the responsibility of some other service (SystemD or Kubernetes or whatever) to notice that your program has died and restart it.  In the event that this process was non-redundant (which is, itself, bad), this may result in some amount of downtime.  However, in BEAM languages this is usually not the case; business logic is typically executed by BEAM worker processes which are isolated from all other BEAM processes (note: BEAM processes do not map to OS processes; read more here: [Erlang processes]).  If one BEAM process crashes, all the other processes keep going[^1].  Furthermore, these processes are usually "supervised" by a [Supervisor] which will detect the crash and start a replacement process, if necessary.

For instance, if you have a web server written in Elixir using the Phoenix framework, each HTTP request that your server receives is handled in its own process.  If you send a request to the server at the same time as I send a request to the server, those requests are processed at the same time[^2].  This is a joy in and of itself, because it means that if my request is generating some big report that takes several seconds, your request doesn't have to wait on mine.  Hooray, throughput!  Anyhow, the BEAM's supervisor pattern also means that if the process generating my report crashes, the process for your request is not at all impacted by that.

_And so_, we arrive at **let it crash**: the idea that since crashing is fairly isolated and the system is (or should be) fault-tolerant, you shouldn't waste your time or clutter up your code with any kind of error-handling code.  If my request to generate that report fails—let's say that the server tried to upload the generated report to S3, but the credentials were wrong—well, we can just let that process crash and our error monitoring service (you *are* using error monitoring, right?[^3]) will report that error to the development team to rectify.  And, in the meantime, the server is still responding to all other requests.

<aside>

There are many _very bad_ ways of handling errors, my least favorite of which is the "cure the symptom" approach.  Let's say you've written some code like this:

```elixir
def build_user(params) do
  %User{
    name: params.name,
	email: params.email
  }
end
```

This is fine until one day, somebody calls this function with `%{email: "example@example.com"}`.  *Now* your function throws this error:

```
** (KeyError) key :name not found in: %{email: "example@example.com"}
    example.ex:8: Example.build_user/1
```

Well, to get this pesky error to go away, we can simple make the `name` key not required!

```elixir
def build_user(params) do
  %User{
    name: params[:name],
	email: params.email
  }
end
```

```elixir
iex> build_user(%{email: "example@example.com"})
%User{name: nil, email: "example@example.com"}
```

Hooray!  All fixed!  _Well_, that is, unless the system expects that users have names at some later time.  _Then_ your system will fail in some place that's not remotely close to `build_user/1` and you'll be really puzzled as to how you wound up with a user without a name to begin with.

This is, coincidentally, the type of issue that a good type system (ML-style like Haskell, OCaml, or Rust) simply won't allow.

Depending on your system, letting the process crash really may have been the right solution (or, otherwise, keep reading).
</aside>

This is all well and good.  However, the *risk* with **let it crash** is:

## Taking It Too Far

I have noticed a general confusion among Elixir developers about what is meant by "let it crash" which sometimes leads applying the concept in places where it doesn't belong.  Recently, I noticed that [the official Elixir documentation] has added [a page about anti-patterns].  This is great!  The work on documenting anti-patterns in Elixir has been going for a couple of years now and the effort has provided me with some good insight.

Under a section on "[non-assertive map access]" (see aside above) was a section on "[non-assertive pattern matching]".  The idea, in summary, is that if data doesn't match some pattern that you're expecting it to have, your function should raise an exception (i.e. probably crash the process).  It's hard to argue with such a concept in general; it was the example that raised my hackles:

```elixir
defmodule Extract do
  def get_value(string, desired_key) do
    parts = String.split(string, "&")

    Enum.find_value(parts, fn pair ->
      key_value = String.split(pair, "=")
      Enum.at(key_value, 0) == desired_key && Enum.at(key_value, 1)
    end)
  end
end
```

OK, this code _is_ problematic, and the authors illustrate why:
```elixir
# Unplanned URL query string format - Unplanned value extraction!
iex> Extract.get_value("name=Lucas&university=institution=UFMG&lab=ASERG", "university")
"institution"   # <= why not "institution=UFMG"? or only "UFMG"?
```

Right, so I'd argue that `"institution=UFMG"` is the correct result here, and coincidentally the standard library agrees:

```elixir
iex> URI.decode_query("name=Lucas&university=institution=UFMG&lab=ASERG")
%{"lab" => "ASERG", "name" => "Lucas", "university" => "institution=UFMG"}
```

_But_, that's not what the authors are arguing here, instead, they write:

> To remove this anti-pattern, get_value/2 can be refactored through the use of pattern matching. So, if an unexpected URL query string format is used, the function will crash instead of returning an invalid value. 

OK, well, we took a tractable problem and determined that the best thing to do was die.  OK, hang on, let's be generous and pretend that this problem is somewhat harder.  Like, what if the query string was just `"university"`?  OK, now _that's_ weird, there's no perceptible key-value structure here at all.  So let's say we crash (by the way, `URI.decode_query` _doesn't_ crash on this input; I'll leave it as an exercise for you to discover what it does).  What now?

> This behavior, shown below, allows clients to decide how to handle these errors and doesn't give a false impression that the code is working correctly when unexpected values are extracted.

See, part of the trouble with this is that there isn't any indication of which functions in your BEAM system throw errors and which don't (not that this is better in many other languages; Java has the `throws` keyword and some fancy effect-handler languages have special bits for this).

[^1]: Unless the process that died was a supervisor, in which case all of its children are also killed (brutal, I know).  There's also the case that if another process tries to contact the process that died, it may itself crash, for better or worse.
[^2]: Give or take, depending on how the BEAM schedules them.  Naturally if the host machine has only one hardware thread available, only one thread of execution can actually take place at a time.
[^3]: One of my previous employers did not have error monitoring for their service, which was and is a cardinal sin.  If you don't have any set up, do yourself a favor and install [Appsignal] (my preference) or [Sentry] (also good and open source).

[Erlang]: https://erlang.org
[Elixir]: https://elixir-lang.org
[Gleam]: https://gleam.run
[Supervisor]: https://www.erlang.org/doc/man/supervisor
[Erlang processes]: https://www.erlang.org/doc/reference_manual/processes
[Appsignal]: https://www.appsignal.com/
[Sentry]: https://sentry.io/
[the official Elixir documentation]: https://hexdocs.pm/elixir/main/
[a page about anti-patterns]: https://hexdocs.pm/elixir/main/code-anti-patterns.html
[non-assertive map access]: https://hexdocs.pm/elixir/main/code-anti-patterns.html#non-assertive-map-access
[non-assertive pattern matching]: https://hexdocs.pm/elixir/main/code-anti-patterns.html#non-assertive-pattern-matching
