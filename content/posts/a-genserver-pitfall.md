---
layout: post
title: Avoiding a GenServer Pitfall
description: An overview of what GenServers do, and things can go awry.
date: 2025-11-01
tags:
- elixir
- erlang
---

# Firstly, what is a GenServer?

GenServer, short for "generic server", is a type of process within BEAM (Erlang, Elixir) applications that can hold state and respond to requests; much like a web server, in a way.  For instance, you could create a GenServer that you could handle requests to store blog posts as well as other requests to list or read those blog posts—but those request would have to be made within the BEAM VM rather than with a web browser.  Within BEAM applications, the two primary purposes of GenServers are to execute code asynchronously and act as a store of shared, mutable state.

## Executing Code Asynchronously

One way to use a GenServer is as a worker.  Let's say that when a new user signs up for your service, you want to send them a welcome email, but you don't want to wait until that email is sent to return a successful account creation HTTP response to the user.  You can send that email asynchronously like so[^1]:

``` elixir
def create_user(user_details) do
  user_details
  |> User.changeset()
  |> Repo.save()
  |> case do
    {:ok, user} ->
      GenServer.cast(MyApp.Mailer, {:send_welcome, user})
      {:ok, user}

    error ->
      error
  end
end
```

``` elixir
defmodule MyApp.Mailer do
  use GenServer

  # For example purposes, we assume the server does not have any meaningful
  # internal state or on-start setup to do.
  def init(_), do: {:ok, nil}

  def handle_cast({:send_welcome, user}, _from, state) do
    user
    |> customize_welcome_email()
    |> @mail_api.send()

    {:noreply, state}
  end
end
```

Since `GenServer.cast` is asynchronous (sends a message and returns immediately, a behavior often termed "fire and forget"), the process invoking `create_user` will not wait for `MyApp.Mailer.handle_cast` to complete.  Thus, control will return from `create_user` likely before the mailer ever tries to send the email.

## Shared, Mutable State

BEAM languages do not, by default, have mutable data structures[^2].  This means that if you have some list `x` and you wish to add an element to it, you will now have a new list `y`.  Also, if `x` exists in Process A, the only way that Process B can read `x` is to receive a message from Process A containing `x`—though, notably, messages passing *copies* the data being sent; Process B does not receive a reference to data inside Process A.

GenServers allow developers to work within these limitations by:
1. Functioning as a stable "location" (name or PID) where the data "changes" over time.
2. Functioning as a shared "location" where multiple processes can manipulate the same data.

One would naturally believe that a mutable data store shared between any number of processes would lead to countless race conditions.  However, many race conditions are mitigated by the fact that GenServers respond to requests sequentially.

For instance, let's imagine that we are creating a seat reservation system for a movie theater.  When a user requests to reserve a seat, we want to:
1. Verify that the requested seat is still available.
2. Process the user's payment.
3. Update the seat in our system to no longer be available.

Consider this GenServer:

``` elixir
defmodule MyApp.SeatReservations do
  @moduledoc """
  A GenServer that keeps track of which seats are reserved.
  """

  use GenServer

  def init(_) do
    {:ok, %{reserved_seats: []}}
  end

  def handle_call({:seat_available?, seat}, _from, state) do
    {:reply, seat in state.reserved_seats, state}
  end

  def handle_call({:mark_reserved, seat}, _from, state) do
    updated_state = Map.update!(state, :reserved_seats, &[seat | &1])

    {:reply, :ok, updated_state}
  end
end
```

Used like so:

``` elixir
def reserve_seat(seat, user) do
  if GenServer.call(MyApp.SeatReservations, {:seat_available?, seat}) do
    process_user_payment(user)

    GenServer.call(MyApp.SeatReservations, {:mark_reserved, seat})

    :ok
  else
    {:error, :taken}
  end
end
```

Using this implementation, a race condition can occur if two users requested the same seat at roughly the same time.  Because the seat is not yet marked as reserved for the first user (payment processing takes time), the second user to request the seat would still see it as available, and both users would ultimately be able to reserve the same seat.

To solve this issue, we can move our ["critical section"] into the GenServer like so:

``` elixir
defmodule MyApp.ReservationServer do
  @moduledoc """
  A GenServer that handles reserving seats.
  """

  use GenServer

  def init(_) do
    {:ok, %{reserved_seats: []}}
  end

  def handle_call({:reserve, seat, user}, _from, state) do
    if seat in state.reserved_seats do
      {:reply, {:error, :taken}, state}
    else
      process_user_payment(user)

      updated_state = Map.update!(state, :reserved_seats, &[seat | &1])

      {:reply, :ok, updated_state}
    end
  end
end
```

And updated usage:

``` elixir
def reserve_seat(seat, user) do
  GenServer.call(MyApp.ReservationServer, {:reserve, seat, user})
end
```

Because GenServers only processes one request at a time, this new GenServer will fully process one request before beginning the next.  This means that if two users request the same seat at about the same time, `reserved_seats` *is* updated when the second user's request begins to be handled since the first user's request must have finished.  Thus the second user will receive an error that the seat is already reserved (i.e. the correct behavior of the system).

# The Pit

The default behavior of `GenServer.call` is to time out after five seconds by default, though the timeout can be set to any number of milliseconds or `:infinity`.  When a `GenServer` fails to respond to a `call` within the timeout, an exception is raised in the calling process.  Unfortunately, getting the timing right in GenServers is *difficult*.  Among the reasons why a GenServer may not respond to a `GenServer.call` before timeout are:
- The callback for your request took too long.  For instance, that callback may be making a call to an external API and their server is running particularly slowly today.
- The GenServer is bogged down handling other requests.  Since requests to GenServers are handled in first-in, first-out (FIFO) order, it is possible that the GenServer *never even got the chance to handle your request before timeout*.
- A combination of the above—the GenServer started handling your request before timeout, but was delayed due to processing other messages first, and was unable to reply before timeout despite the callback's runtime being less than `timeout` milliseconds.

One can easily imagine a situation where, using the code above, the payment processing step takes more than five seconds at least some of the time.

**There is no way to totally avoid this class of issue**.  Even a GenServer whose callbacks execute in microseconds can become bogged down by a sudden surge of requests.

In the last month, I've experienced two different GenServer timing related production issues:
- A GenServer callback that *never* terminated because a library function called in my callback evidently doesn't ever terminate sometimes.  The GenServer was effectively dead, but never restarted because it wasn't *actually* dead.
- A GenServer callback that *sometimes* takes a while (it's calling an external API) and thus on those occasions later GenServer calls are timing out.  The system took a while to recover.

# A Partial Solution

While this problem cannot be completely resolved, there is one way to constrain the runtime of GenServer callbacks and thus reduce the risk of timeouts: [Tasks].  Ideally, GenServers would have a built-in way to limit the allowable time for a GenServer callback's execution.  Since there *isn't*, we can can make one ourselves:

``` elixir
def handle_call(:foo, _from, state) do
  task =
    Task.async(fn ->
      slow_function(state)
    end)

  case Task.yield(task, 4_000) || Task.shutdown(task) do
    {:ok, result} ->   {:reply, {:ok, result}, state}
    {:exit, reason} -> {:reply, {:error, reason}, state}
    nil ->             {:reply, {:error, :timeout}, state}
  end
end
```

Let's break this down.

``` elixir
  task =
    Task.async(fn ->
      slow_function(state)
    end)
```

The first thing this callback does is begin a Task running `slow_function`.  Here, `slow_function` is a placeholder for the work that is to be done in this GenServer's callback; everything else in this callback is about limiting the runtime (we'll reduce the boilerplate in a moment).

``` elixir
  case Task.yield(task, 4_000) || Task.shutdown(task) do
    {:ok, result} ->   {:reply, {:ok, result}, state}
    {:exit, reason} -> {:reply, {:error, reason}, state}
    nil ->             {:reply, {:error, :timeout}, state}
  end
```

The expression `Task.yield(task, 4_000) || Task.shutdown(task)` gives the task running `slow_function` (i.e. the callback's "real" work) 4,000ms to complete.  The value of 4,000ms is somewhat arbitrary; but the central idea is that:
1. It should probably be less than the default `GenServer.call` timeout of 5,000ms unless every caller is using a longer, custom timeout
2. It should probably leave some headroom to account for the fact that the `GenServer.call` timeout may have started counting down while a previous callback was being processed, and so this callback has *less* than the full 5,000ms to respond.

You can tweak this timeout based on your system's needs.

`Task.yield` and `Task.shutdown` have conveniently have the same return types:
- `{:ok, reply}`: The Task completed and returned a value.
- `{:exit, reason}`: The Task died unexpectedly.
- `nil`: No reply was received.

Here we're using `||` to essentially say "If the Task hit its time limit, shut it down".  Based on which result we receive here, we produce a corresponding reply for our GenServer.

OK, as promised, one can reduce the boilerplate by defining a helper function:

``` elixir
def time_limited(fun, time_limit) do
  task = Task.async(fun)

  Task.yield(task, time_limit) || Task.shutdown(task)
end
```

Used like so:

``` elixir
def handle_call(:foo, _from, state) do
  case time_limited(fn -> slow_function(state) end, 4_000) do
    {:ok, result} ->   {:reply, {:ok, result}, state}
    {:exit, reason} -> {:reply, {:error, reason}, state}
    nil ->             {:reply, {:error, :timeout}, state}
  end
end
```

To cut down boilerplate even further, you can customize this for GenServers (create the `{:reply, ..., ...}` tuples inside `time_limited`) or write a macro.

One caveat to note here is that spawning a Task is going to consume additional resources—most notably memory.  As noted above, when one process sends a message to another, the data in that message is *copied* between the processes.  This also applies to spawning new processes: the data you are spawning the new process with is copied into that process.  Consider a typical GenServer interaction with the above pattern:
1. Process P sends a message to GenServer G
2. G spawns Task T.

Let's conjecture that P is sending data needed to fulfill the GenServer's callback.  Therefore:
- The original data exists in P
- The data is copied from P to G with `GenServer.call`
- The data is copied from G to T with `Task.async`

Therefore there are three total copies of this data.  If the data in question is small, such as an atom or integer, this is not of much concern.  *However*, if the data is larger, say a struct with a large number of fields or nested structures, creating two additional copies of that data may consume more memory than you would be comfortable allocating.

It is, as a general note, a best practice to send as little data in messages as possible for this reason (e.g. send an ID that can be used to fetch a struct rather than the struct itself).

# In Sum

You can mitigate the problem of long-running GenServer callbacks causing timeouts within your system by wrapping the functionality of the callback in a time-limited Task.  To this end, I believe that one should generally *default* to time-limiting GenServer callbacks, unless you have a good reason not to[^3].  It's a bit of boilerplate and additional memory overhead, but when used thoughtfully it'll save you from production headaches in the long run.

[^1]: Generally speaking, you should wrap `GenServer.cast` and `GenServer.call` invocations in another function, e.g.
    ``` elixir
    def send_welcome_email(user) do
      GenServer.cast(MyApp.Mailer, {:send_welcome, user})
    end
    ```
    The rationale for this indirection is that the implementations for these types of operations are often subject to change.  For instance, sending the welcome email might later be performed by a [Task] instead.  If that were the case, and our application is sending emails via this `send_welcome_email/1` function, we wouldn't need to update everywhere that sends an email to use a Task, we'd only need to update `send_welcome_email/1`.
[^2]: You can make a mutable data structure using natively implemented functions ("NIFs"), but this is not commonly done.
[^3]: There are some use cases for GenServer to run time-unbounded callbacks, such as async workers.  It's a somewhat niche use case, however, since if the GenServer crashes for any reason (or the node goes down), the GenServer's work queue is lost.  A better option would be to opt for a persistence-backed worker such as [Oban].

["critical section"]: https://en.wikipedia.org/wiki/Critical_section
[Task]: https://hexdocs.pm/elixir/Task.html
[Tasks]: https://hexdocs.pm/elixir/Task.html
[Oban]: https://hexdocs.pm/oban/Oban.html
