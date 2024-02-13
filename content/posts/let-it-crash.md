---
title: "Let it Crash?"
layout: post
date: 2024-02-12
description: While we're busy letting things crash, let's not forget about our users.
tags:
- elixir
- erlang
- software development
---

![The oceanliner Titanic sinking in the North Atlantic](/images/titanic.webp)

In the BEAM ecosystem ([Erlang], [Elixir], [Gleam], etc), there's a common idiom:

## Let It Crash

And your first reaction to this is probably astonishment, because crashing is, generally-speaking, not ideal.  In traditional, single-threaded systems, a crash means that your program dies and—in server settings—it's then the responsibility of some other service (SystemD or Kubernetes or whatever) to notice that your program has died and restart it.  In the event that this process was non-redundant (which is, itself, bad), this may result in downtime.  However, in BEAM languages this is usually not the case; business logic is typically executed by BEAM worker processes[^1] which are isolated from all other BEAM processes.  If one BEAM process crashes, all the other processes keep going[^2].  Furthermore, these processes are usually "supervised" by a [Supervisor] which will detect the crash and start a replacement process, if necessary.

If you have a web server written in [Phoenix], the Elixir server framework, each HTTP request that your server receives is handled in its own process.  Ergo, if you send a request to the server at the same time as I send a request to the server, those requests are processed at the same time[^3], in different processes.  This is a joy in and of itself, because it means that if my request is generating some big report that takes several seconds, your request doesn't have to wait on mine.  Hooray, throughput!  The BEAM's process isolation also means that if the process generating my report crashes, the process for your request is not impacted.

_And so_, we arrive at **"let it crash"**: the idea that since crashing is fairly isolated and the system is (or should be) fault-tolerant, you shouldn't waste your time or clutter up your code with any kind of error-handling.  If my request to generate that report fails—let's say that the server tried to upload a file to S3 but the credentials were wrong—well, we can just let that process crash and our error monitoring service (you *are* using error monitoring, right?[^4]) will report that error to the development team to rectify.  And, in the meantime, the server is still responding to all other requests (like yours).

<aside>

#### Treating the Symptom

By contrast, defensive programming leads to many _very bad_ ways of handling errors, my least favorite of which is the "treat the symptom" approach.  Let's say you've written some code like this:

```elixir
def build_user(params) do
  %User{
    name: params.name,
	email: params.email
  }
end
```

This is fine until one day, somebody calls this function with `%{email: "example@example.com"}`.  *Now* your function throws this error:

```elixir
** (KeyError) key :name not found in: %{email: "example@example.com"}
    example.ex:8: Example.build_user/1
```

To get this pesky error to go away, we can simply make the `name` key not required!

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

Voila!  All fixed!  _Well_, that is, unless the system expects that users have names at some later time.  _Then_ your system will fail in some place that's not remotely close to `build_user/1` and you'll be really puzzled as to how you wound up with a user without a name to begin with.

Depending on your system, letting the process crash on the invalid input really may have been the right solution (or, otherwise, keep reading).
</aside>

This is all well and good.  However, the *risk* with **let it crash** is:

## Taking It Too Far

I have noticed a general confusion among Elixir developers about what is meant by "let it crash" which sometimes leads applying the concept in places where it doesn't belong.

Recently, I noticed that [the official Elixir documentation] has (as of version 1.16.0) added [a page about anti-patterns].  This is great!  The work on documenting anti-patterns in Elixir has been going for a couple of years now and the effort has provided me with valuable insights.

My umbrage with this documentation arose from a section on "[non-assertive pattern matching]" (under a section on "[non-assertive map access]"; see aside above).  The idea is, in summary, that if data passed to a function doesn't match some pattern that you're expecting it to have, your function should raise an exception (i.e. probably crash the process[^5]).  It's hard to argue with such a concept in general; it was the example that raised my hackles:

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

We took a tractable problem and determined that the best thing to do was die.  OK, hang on, let's be generous and pretend that this problem is somewhat harder.  Like, what if the query string was just `"university"`?  OK, now _that's_ weird, there's no perceptible key-value structure there at all.  So let's say we crash (by the way, `URI.decode_query` _doesn't_ crash on this input; I'll leave it as an exercise for you to discover what it does)—what now?

> This behavior, shown below, allows clients to decide how to handle these errors and doesn't give a false impression that the code is working correctly when unexpected values are extracted.

See, part of the trouble with this idea is that there isn't any indication of which functions in your BEAM system throw errors and which don't[^6].  Unless you're wrapping every function call in the sparsely-used [`try...catch`] construct, **every** call introduces the risk of a potential crash which makes the framing "allows the client to decide how to handle these errors" a bit rich.

"OK, so what if you crash?" you ask.  "Isn't that an acceptable outcome?"  Well the "so what" is that we need to return a response to the HTTP call!  If a user's browser sends off a request and doesn't get some response back, it'll sit there spinning until it eventually gives up which makes for some really terrible UX.

Phoenix helps us out a little bit here.  By default, if the process handling a request crashes, Phoenix will detect that crash and return a canned HTTP 500 response to the user with a message to the effect of "Something went wrong".  This is all well and good except that _in this example it's wrong_.  The client sent us some data in an invalid format, which makes this failure *their fault, not ours*.  This isn't just an exercise in passing blame, the point I'm trying to make is that *the HTTP 4XX status codes exist to convey that the client's request was unacceptable and we're not using them*.

Some years ago, I was building an API integration for a 3rd party vendor with a sparsely-documented API.  The biggest hurdle in building this integration was that if my request to the service was in any way incorrect (and seeing as it was sparsely documented, this was frequently the case), the service would just return a canned HTTP 500 response: "Something went wrong."  Well, OK, *sure*, but it's my fault and I'd really like to know what exactly it was I did wrong, please.  "*Something went wrong.*"

Don't develop this kind of API.  After all, the most common case is that your API is being consumed by some internal team building a front-end in React or Vue or something, and they might (rightly) toss rotten vegetables at you in the cafeteria if you make their lives unnecessarily difficult.  Your API may, alternatively, be consumed by some external client who will send you passive-aggressive emails when they can't make heads or tails of it.

## Try to Help Your Users

In the years I've been working in this industry, the biggest failure I've witnessed is developers totally lacking empathy for the users of their software.  I'm disheartened to say that this doesn't seem to be getting better—in fact, it's likely getting worse.  Have you seen the AWS Developer Console lately?  I've got a whole separate rant on this that I'm going to break out into another post.

So, now that we (hopefully) understand the issue, what can be done differently?

## Handle Some Cases

Specifically, I bundle these cases into a neat little package called "the application boundary".  The part of your application where it touches something that is not part of your application is the boundary, and anything that comes across that boundary should not be trusted.

HTTP requests coming into your BEAM server are a great example of data crossing the application boundary.  Despite appearances, these requests could be coming from *anywhere*.  They *could* be sent from a well-intentioned person browsing your site using the legitimate front-end, or they could be sent by a malicious actor using `curl`.  A request could be well-formatted, or it could be nonsense.  The only way you'll know is to check, and the best place to check is right there at the application boundary; as early in the request cycle as you can manage.

There are a few libraries that facilitate data integrity checks, such as [Params], [Drops], and [Vex].  [Using Ecto schemaless changesets] works also, and (most likely) allows you to avoid adding another dependency[^7].  It makes sense to perform data integrity checks in your controllers, though if you can perform them in a Plug earlier in your chain (a la your authentication Plug) that's even better.

Here's the salient portion: When our application detects that a request is malformed, our response should use a HTTP 4XX status code.  And, if we're feeling charitable (and hopefully we are), we should communicate what, exactly, the issue was.  If the user's query string contained some kind of invalid key-value pair, wouldn't it be nice if we told them so?  Sure beats **"Something went wrong"**.

[^1]: BEAM processes do not map to OS processes; they are "[green threads]".  Read more about them here: [Erlang processes]
[^2]: Unless the process that died was a supervisor, in which case all of its children are also killed (brutal, I know).  There's also the case that if another process tries to contact the process that died, it may itself crash, for better or worse.
[^3]: Give or take, depending on how the BEAM schedules them.  Naturally if the host machine has only one hardware thread available, only one thread of execution can actually take place at a time.
[^4]: One of my previous employers did not have error monitoring for their service, which was and is a cardinal sin.  If you don't have any set up, do yourself a favor and install [AppSignal] (my preference) or [Sentry] (also good and open source).
[^5]: Elixir does have a [`try...catch`] construct, but it's very sparingly used.
[^6]: Not that this is better in many other languages—Java has the `throws` keyword for function annotation, [Koka has "effect handlers"], and [Unison has "abilities"], but they are the exceptions (pun not intended), not the rule.
[^7]: Technically we could implement this data validation pattern with crashing and `catch`ing, though that seems to me to be violating a different anti-pattern: [Exceptions for control-flow]; the example for which is, curiously, taking some code that raises an exception and making it not raise an exception, in a complete inversion of the [non-assertive pattern matching] example.  Charitably, I think the salient difference is that the former example uses `catch`, and so the point is really "don't use `catch`" which, happily, most BEAM programmers don't already.

[Erlang]: https://erlang.org
[Elixir]: https://elixir-lang.org
[Gleam]: https://gleam.run
[green threads]: https://en.wikipedia.org/wiki/Green_thread
[Supervisor]: https://www.erlang.org/doc/man/supervisor
[Erlang processes]: https://www.erlang.org/doc/reference_manual/processes
[Phoenix]: https://phoenixframework.org/
[Appsignal]: https://www.appsignal.com/
[Sentry]: https://sentry.io/
[the official Elixir documentation]: https://hexdocs.pm/elixir/main/
[a page about anti-patterns]: https://hexdocs.pm/elixir/main/code-anti-patterns.html
[non-assertive map access]: https://hexdocs.pm/elixir/main/code-anti-patterns.html#non-assertive-map-access
[non-assertive pattern matching]: https://hexdocs.pm/elixir/main/code-anti-patterns.html#non-assertive-pattern-matching
[Koka has "effect handlers"]: https://koka-lang.github.io/koka/doc/book.html#why-handlers
[Unison has "abilities"]: https://www.unison-lang.org/docs/fundamentals/abilities/error-handling/
[`try...catch`]: https://hexdocs.pm/elixir/Kernel.SpecialForms.html#try/1
[Params]: https://hexdocs.pm/params
[Drops]: https://hexdocs.pm/drops
[Vex]: https://hexdocs.pm/vex
[Ecto]: https://hexdocs.pm/ecto
[Using Ecto schemaless changesets]: https://blog.appsignal.com/2023/11/07/validating-data-in-elixir-using-ecto-and-nimbleoptions.html#using-ecto
[Exceptions for control-flow]: https://hexdocs.pm/elixir/design-anti-patterns.html#exceptions-for-control-flow
