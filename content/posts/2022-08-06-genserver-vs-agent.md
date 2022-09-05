---
layout: post
title: "Elixir GenServers vs Agents"
date: 2022-08-06
description: Elixir, and the OTP ecosystem it is built upon, offer a variety of different kinds of processes.  Two of the most common are GenServer and Agent, which are similar but have different applications.
tags:
- elixir
---

Just as [Protocols are an abstraction built upon Behaviours], Agents are an abstraction built upon GenServers.  While GenServers can serve a wider variety of purposes, Agents serve a single purpose: storing state.

GenServer
---------

The [Elixir documentation for GenServer] summarizes them as such:
> A GenServer is a process like any other Elixir process and it can be used to keep state, execute code asynchronously and so on. The advantage of using a generic server process (GenServer) implemented using this module is that it will have a standard set of interface functions and include functionality for tracing and error reporting. It will also fit into a supervision tree.

Included in this summary are two potential use-cases for GenServers:
1. Keep state
2. Execute code asynchronously

Let's explore each of these.

### Keeping State

When the documentation refers to "keeping state," it means between processes.  In Elixir, data structures cannot (at least not without some trickery) be shared between processes; each process has it's own data, and the only way to share data between processes is by passing messages (e.g. using [`send/2`]).  This frees Elixir developers from having to worry about race conditions much of the time, but it also creates a new question of how to allow multiple processes within an Elixir system to have read/write access to a shared piece of data.  I have included an example for doing this with both GenServers and Agents in the latter half of this post.

### Executing Code Asynchronously

The idea here is simple: imagine a web request process wants a report to be generated and an email sent, but also doesn't want to have to wait until those are finished before sending a response to the user.

Enter GenServer.  When an application has a dedicated GenServer process to perform a specific task, other processes can request the GenServer to perform its task by sending it a message, and then continue on with their other work[^4].

While doing this with Agent _is possible_ it's not the intended use of Agent, and so should be avoided.

Agent
-----

[The Elixir documentation for Agent] summarizes them simply as:

> Agents are a simple abstraction around state.

The commentary around "Keeping State" from the GenServer section above applies here, with the noted caveat that Agents are _optimized_ for this use case.

Feature Comparison
------------------

Agents surface much, but not all, of the functionality from their underlying GenServers.

| Feature                                              | GenServer Support | Agent Support |
|------------------------------------------------------|-------------------|---------------|
| call (send message and await response)               | Yes               | Yes           |
| cast (send message without waiting for response)     | Yes               | Yes           |
| info (receive arbitrary messages sent with `send/2`) | Yes               | No            |
| continue (init function can continue async)          | Yes               | No            |
| terminate (custom behavior on shutdown)              | Yes               | No            |
| code change (custom code update handling)            | Yes               | [Sort-of]     |

[Sort-of]: https://hexdocs.pm/elixir/1.13.4/Agent.html#module-hot-code-swapping

Additionally, the `Agent` module provides a variety of [`Access`]-like functions that help the user think of it as a datastructure instead of a process, such as:

- [`get/3`], [`get/5`]
- [`update/3`], [`update/5`]
- [`get_and_update/3`], [`get_and_update/5`]

[`Access`]: https://hexdocs.pm/elixir/1.13.4/Access.html
[`get/3`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#get/3
[`get/5`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#get/5
[`update/3`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#update/3
[`update/5`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#update/5
[`get_and_update/3`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#get_and_update/3
[`get_and_update/5`]: https://hexdocs.pm/elixir/1.13.4/Agent.html#get_and_update/5

The utility of these, and the advantages they provide over coding the same functionality with a GenServer, is best shown through an example.

Example
-------

Let's say that we're creating a bulletin board where users can post new messages.  Each request that comes into the bulletin board system is either posting a new message or reading the messages that have been posted.

![A bulletin board with notes on it](/images/bulletin_board.jpg)

With Phoenix, as well as with other Elixir/Erlang web frameworks, each request is handled in its own process.  Thus, the request-handling process' state will only include a few things that have been given to it—most likely _not_ including our repository of messages.  These request processes need to be able to read or write to a shared repository of messages.

Let's create a GenServer to hold the messages[^1]!

Conventionally, the first place to start is by defining a module and calling `use GenServer` within it:

```elixir
defmodule BBS.MessageRepository do
  use GenServer
```

Next, we'll define our server functionality.  This consists of an `init/1` function which is invoked when the server starts up and a `handle_call/3` function which is invoked when the server receives a message[^2].

```elixir
  # Server

  def init(messages) do
    {:ok, messages}
  end

  def handle_call({:post, message}, _from, messages) do
    {:reply, :ok, [message | messages]}
  end

  def handle_call(:read_all, _from, messages) do
    {:reply, messages, messages}
  end
```

Our `init/1` allows the server to be initialized with a list of messages, which we use as the initial state.  The return value `{:ok, messages}` indicates that the server started successfully and we want our GenServer's state to be `messages`.

Our `handle_call/3` function can handle two kinds of incoming messages:
1. `{:post, message}`, which requests that `message` be posted to the bulletin board
2. `:read_all`, which requests all of the messages on the bulletin board

The return value in both cases takes the form `{:reply, response, new_state}`.  There are other allowable formats for return values of `handle_call/2`, but this is the only one we need for this example.

If we ended our module here, a client could use this code like so:
```elixir
iex(1)> {:ok, pid} = GenServer.start_link(BBS.MessageRepository, ["Welcome to My Bulletin Board!"])
{:ok, #PID<0.163.0>}
iex(2)> GenServer.call(pid, {:post, "Block Party!"})
:ok
iex(3)> GenServer.call(pid, :read_all)
["Block Party!", "Welcome to My Bulletin Board!"]
```
_Your PID will almost certainly be different—this is to be expected!_

This isn't very convenient, though, and each request process shouldn't call `GenServer.start_link` start it's own server—that would defeat the purpose!—they need to share one server.

Let's fix this by configuring our application start the server.  If your Elixir app was generated with a supervision tree (i.e. you passed `--sup` to `mix new` or are using Phoenix/another framework), you should have an `application.ex` file, e.g. `lib/bbs/application.ex`.  This file is where the application's supervision tree is defined, and processes specified in the list of children will be started when your application boots.  Let's add our message repository as a child in this list.

```elixir
    children = [
      %{
        id: BBS.MessageRepository,
        start:
          {GenServer, :start_link,
           [
             BBS.MessageRepository,
             ["Welcome to My Bulletin Board!"],
             [name: BBS.MessageRepository]
           ]}
      }
    ]
```

This is a bit verbose, but we'll clean it up later.  The key concept here is that the `children` list should be reducible to a list of maps that at least have an `:id` key—whose value is used internally by the supervisor to identify this child—and a `:start` key—whose value is a Module-Function-Arguments (MFA) tuple that indicates what function to call to start the process.

You may have noticed that `[name: BBS.MessageRepository]` was added as a third argument to `GenServer.start_link`.  When `:name` refers to an atom (and in Elixir, modules are atoms), that atom is registered as the node-local name for the process.  With the name registered, we can use the name instead of the PID when invoking `GenServer.call/2` (and other process communication functions such as `send/2`).  By convention, the name of the module that defines the process is used as the process name.

With this in place, our application will start a shared message repository which we can immediately access by its name:

```elixir
iex(1)> GenServer.call(BBS.MessageRepository, {:post, "Block Party!"})
:ok
iex(2)> GenServer.call(BBS.MessageRepository, :read_all)
["Block Party!", "Welcome to My Bulletin Board!"]
```

This works pretty well.  However, it's conventional in Elixir to use wrapper "client" functions instead of calling GenServer functions directly from your application code.  This is especially important when data flowing from your application into the GenServer or vice versa require a bit of processing.  That's not the case for us, but we'll define client functions anyways for the sake of convention.  We'll put these below the server functions in the `BBS.MessageRepository` module[^3].

```elixir
  # Client

  def post(message) do
    GenServer.call(BBS.MessageRepository, {:post, message})
  end

  def read_all() do
    GenServer.call(BBS.MessageRepository, :read_all)
  end
```

Now, from anywhere in our application we can simply invoke `BBS.MessageRepository.post("Block Party!")` to post a message about a block party and `BBS.MessageRepository.read_all()` to read all existing messages.  Not too shabby!

You can view [this first iteration of `BBS.MessageRepository` on GitHub](https://github.com/J3RN/BBS/blob/55d0215ec0e2c583fc2e2d92cd241653ac59a942/lib/bbs/message_respository.ex).

Lastly, the cleanup I mentioned before.  At the very bottom of our `BBS.MessageRepository` module we can define a `start_link/1` function like so:

```elixir
  def start_link(messages) do
    GenServer.start_link(
      BBS.MessageRepository,
      messages,
      name: BBS.MessageRepository
    )
  end
end
```

The key incentive to doing this is that it allows us to simplify the specification for this process in the children list of the application supervisor:

```elixir
    children = [
      {BBS.MessageRepository, ["Welcome to My Bulletin Board!"]}
    ]
```

Additionally, moving the logic regarding how the process starts into the module where other aspects of process are defined lowers the coupling from our application module to our process module.  For instance, if we wanted to change how the `BBS.MessageRepository` process was started (*cough* foreshadowing *cough*), we would only have to update the `BBS.MessageRepository` module.

You can view [this slightly expanded `BBS.MessageRepository`](https://github.com/J3RN/BBS/blob/d5e2f7a5677a8e37b4d6a8e02c1574cfcfe4c5cc/lib/bbs/message_respository.ex) and [the terser `application.ex`](https://github.com/J3RN/BBS/blob/d5e2f7a5677a8e37b4d6a8e02c1574cfcfe4c5cc/lib/bbs/application.ex) on GitHub.

<aside>

#### How Does Specifying a Child with a Tuple Work?

Unfortunately, it's a bit complicated.

Per [the Supervisor documentation], children can be specified by:
- A map, as discussed previously
- A module
- A `{module, arg}` tuple, as used above
- A six-element tuple (deprecated)

The list of child specifications is passed to `Supervisor.start_link/2`, where they are passed to `Supervisor.init/2`, and subsequently to `Supervisor.init_child/1` (private), which is responsible for turning each of these into the previously discussed map format (except for the sextuple format, but I'm ignoring that because it's deprecated).

When `Supervisor.init_child/1` is given a map, its work is already complete and the map is returned without any changes.  When only a module is given, it is wrapped in a tuple with an empty list and passed again to `Supervisor.init_child/1`. When a tuple containing a module and some second element ("the argument") is given, `Supervisor.init_child/1` assumes that the module defines a `child_spec/1` function that, when called with the argument, will return the desired map.

"But wait," you say, "we didn't define a `child_spec/1` function in `BBS.MessageRepository`!"  You're correct.  However, when we invoke `use GenServer` at the top of the `BBS.MessageRepository` module, [`GenServer`'s `__using__` macro] is invoked, which defines a `child_spec/1` for us.  This generated `child_spec/1` function returns a map that specifies the module's name as the `:id`—which we did before, so that's fine—and refers to a local `start_link/1` function for the `:start` key.  Thus, we needed to define a `start_link/1` function to start our server process, which we did above.

You can override this generated `child_spec/1` function by simply defining such a function within the module body, if needed.

This functionality allows us to avoid some boilerplate when specifying children processes for a supervisor and avoid the aforementioned coupling, but comes at the cost of being a bit convoluted when you're trying to figure out how it works.

</aside>

This wraps up a basic example of storing state with a GenServer.

Let's refactor our `BBS.MessageRepository` module to use Agent instead of GenServer.  Firstly, we'll change `use GenServer` to `use Agent`:

```elixir
defmodule BBS.MessageRepository do
  use Agent
```

Easy enough! Next, we'll need to change our `init/1` function very subtly:

```elixir
  # Server

  def init(messages) do
    messages
  end
```

Did you notice the change? We changed the return value from `{:ok, messages}` to just `messages`.  Agents don't appear to have a method to specify that the server initialization failed, whereas GenServers' `init/1` may return `:ignore` or `{:stop, reason}` to indicate that the server should not be started.  Regardless, this isn't a problem for us in this example.

Next, we're going to convert our `handle_call/2` function clauses into two new functions: `handle_post/2` and `handle_read_all/1`. You'll see why this is necessary soon.

```elixir
  def handle_post(messages, message) do
    [message | messages]
  end

  def handle_read_all(messages) do
    messages
  end
```

Again, we've been able to dispose of some syntactic noise—we don't take `from` as an argument (which we didn't use anyways) and don't have to wrap our return values in `{:reply, response, new_state}` tuples. Speaking of which, something interesting to note is that `handle_post/2` returns only the new state and `handle_read_all/1` only returns a response for the client (which is coincidentally also the state here, but doesn't need to be).  These functions also now take different numbers of arguments!  These changes are facilitated by the `Agent` functions we'll be using in our updated client functions.

```elixir
  # Client

  def post(message) do
    Agent.update(BBS.MessageRepository, BBS.MessageRepository, :handle_post, [message])
  end

  def read_all() do
    Agent.get(BBS.MessageRepository, BBS.MessageRepository, :handle_read_all, [])
  end
```

In `post/1`, we make use of `Agent.update` which takes the name of a running Agent as the first argument—we're still naming the process after the module, per convention—and a module, function, and arguments for the next three arguments (essentially an MFA tuple, discussed earlier).  The specified function will receive the Agent's state as its first argument and the arguments specified in the `Agent.update` call as subsequent arguments.  The return value of this function will be the new state for the Agent.  `Agent.update` always returns `:ok`.

The implementation of `read_all` is similar, except that the return value of the function specified to `Agent.get` will be the return value of `Agent.get` and has no impact on the Agent's state.

A nicety of this approach is the ability to assign arbitrary names to the handler functions, whereas with GenServer they had to be clauses of the `handle_call` function.

Lastly, let's update our `start_link/1` function:

```elixir
  def start_link(messages) do
    Agent.start_link(BBS.MessageRepository, :init, [messages], name: BBS.MessageRepository)
  end
end
```

This didn't change all that much; we swapped "GenServer" for "Agent", now have to specify that we want the `init` function (which was implicit before), and also need to wrap `messages` in a list to make our first three arguments essentially and MFA tuple.  A side effect of this is that we could now pass additional arguments to `init`, if we wanted, or even renamed `init`, but that's not necessary for this example.

<aside>

You can view [the entire module with Agent on GitHub](https://github.com/J3RN/BBS/blob/fc4501ec20d4886505df76bc940716909a95b689/lib/bbs/message_respository.ex).

#### Client/Server Separation

Being able to specify the module for the handler functions makes it pretty easy to create a dedicated server module.  For instance, we could move our server functions (`handle_post/2`, `handle_read_all/1`, and `init/1`) into a new `BBS.MessageRepository.Server` module and make that module's name the second argument in the `Agent.get` and `Agent.update` function calls above and the first argument in `Agent.start_link`. _Voila!_  With these changes, we would have a clear separation of what code runs inside the Agent process from what code runs in the calling (or client) process.

We also could have split the server functions from the client functions with our GenServer implementation, but it would be more involved.

</aside>

If you can believe it, this code can become even more concise!  Instead of passing a module, function, and arguments to the `Agent` functions, we can pass an anonymous function instead.  Here, we'll combine our client and server functions such that the server portion is just an anonymous function inside the client functions:

```elixir
defmodule BBS.MessageRepository do
  use Agent

  def post(message) do
    Agent.update(BBS.MessageRepository, fn messages ->
      [message | messages]
    end)
  end

  def read_all() do
    Agent.get(BBS.MessageRepository, fn messages ->
      messages
    end)
  end

  def start_link(messages) do
    Agent.start_link(fn -> messages end, name: BBS.MessageRepository)
  end
end
```

That's it!  Those 19 lines of code constitute our entire message repository process!  This level of brevity is not available with GenServer, only with Agent.

If you'd like to see [this terser Agent version on GitHub](https://github.com/J3RN/BBS/blob/c679c7c3c07703901d2ee0ef275ff91757ef00dd/lib/bbs/message_respository.ex), that's also an option.

The downside to this condensed code, in my opinion, is that the separation between what code runs in the server process from what code runs in the client process is at its fuzziest.  I can imagine confusing what runs where when giving this code a quick skim.

Conclusion
----------

GenServers and Agents are both kinds of processes defined within Elixir.  The former allows for great flexibility and different use cases whereas the latter is optimized an implementation of the former for allowing processes to read/write access to a single piece of data.

I hope that this post has given you the knowledge to make the right choice for your application!

[^4]: The process mailbox acts like a FIFO queue; messages sent first will be processed first.  However it's worth noting that if the receiving process dies for any reason, it's mailbox—and thus queue of work—will be lost.

[^1]: This works well for prototyping, but you should be aware that when the GenServer process dies, all the data that it was holding will be lost. For a more persistent data store, you'll want to use either [DETS] or an external database such as Redis or PostgreSQL.

[^2]: There are other kinds of messages that can be received by a GenServer—including cast messages and info messages, handled by [`handle_cast/2`] and [`handle_info/2`], respectively—but I only cover call messages in this post.

[^3]: I'm not a fan of having client functions and server functions in the same file, especially if they contain any amount of logic, but this is the Elixir convention.  Dave Thomas explores this in his talk ["I Write Bad Elixir. So Do You!"].  Humorously `Agent` bucks the convention and splits client and server, with the `Agent` module containing the client functions and the `Agent.Server` module containing the server functions.

[Protocols are an abstraction built upon Behaviours]: /posts/2021-10-14-elixir-behaviours-vs-protocols
[Elixir documentation for GenServer]: https://hexdocs.pm/elixir/1.13.4/GenServer.html
[The Elixir documentation for Agent]: https://hexdocs.pm/elixir/1.13.4/Agent.html
[`send/2`]: https://hexdocs.pm/elixir/1.13.4/Kernel.html#send/2
[DETS]: https://www.erlang.org/doc/man/dets.html
[`handle_cast/2`]: https://hexdocs.pm/elixir/GenServer.html#c:handle_cast/2
[`handle_info/2`]: https://hexdocs.pm/elixir/GenServer.html#c:handle_info/2
["I Write Bad Elixir. So Do You!"]: https://www.youtube.com/watch?v=6U7cLUygMeI
[the Supervisor documentation]: https://hexdocs.pm/elixir/1.13.4/Supervisor.html#module-child_spec-1
[`GenServer`'s `__using__` macro]: https://github.com/elixir-lang/elixir/blob/v1.13.4/lib/elixir/lib/gen_server.ex#L742-L843
