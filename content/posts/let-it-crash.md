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

And your first reaction to this is probably astonishment, because crashing is, generally-speaking, not ideal.  In traditional single-threaded systems—in server settings—a crash means that your program dies and it's the responsibility of some other service (SystemD or Kubernetes or whatever) to notice that your program has died and restart it.  In the event that this process was non-redundant (which, itself, bad), this may result in some amount of downtime.  However, in BEAM languages this is usually not the case; business logic is typically executed by a BEAM worker process which is isolated from all other BEAM processes (note: BEAM processes do not map to OS processes; read more here: [Erlang processes]).  If one BEAM process crashes, all the other processes keep going[^1].  Furthermore, these processes are usually "supervised" by a [Supervisor] which will detect the crash and start a replacement process, if necessary.

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



[^1]: Unless the process that died was a supervisor, in which case all of its children are also killed (brutal, I know).  There's also the case that if another process tries to contact the process that died, it may itself crash, for better or worse.
[^2]: Give or take, depending on how the BEAM schedules them.  Naturally if the host machine has only one hardware thread available, only one thread of execution can actually take place at a time.
[^3]: One of my previous employers did not have error monitoring for their service, which was and is a cardinal sin.  If you don't have any set up, do yourself a favor and install [Appsignal] (my preference) or [Sentry] (also good, and open source!).

[Erlang]: https://erlang.org
[Elixir]: https://elixir-lang.org
[Gleam]: https://gleam.run
[Supervisor]: https://www.erlang.org/doc/man/supervisor
[Erlang processes]: https://www.erlang.org/doc/reference_manual/processes
[Appsignal]: https://www.appsignal.com/
[Sentry]: https://sentry.io/welcome/
