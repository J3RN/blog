---
layout: post
title: "Elixir Behaviours vs Protocols"
date: 2021-10-14T22:40:42-04:00
draft: true
tags:
- elixir
---

As developers begin learning Elixir, they often have a very understandable confusion around the difference between Elixir's [behaviours](https://hexdocs.pm/elixir/typespecs.html#behaviours) and [protocols](https://hexdocs.pm/elixir/Protocol.html). These are similar constructs, but with important differences.

## Behaviours

Behaviours are conceptually quite simple:

> A behaviour module defines a set of functions and macros (referred to as callbacks) that callback modules implementing that behaviour must export.

A commonly use behaviour is GenServer, used like so:

```elixir
defmodule MyApp.APICache do
  @behaviour GenServer
end
```

However, this module fails to compile with the following error:

```
warning: function init/1 required by behaviour GenServer is not implemented (in module MyApp.APICache)
```

In other words, the GenServer behaviour specified that modules implementing it _must_ define an `init/1` function. The rationale behind behviours is to allow modules to be _interchangeable_. By enforcing the existence of an `init/1` function, the GenServer behaviour ensures that if a module knows how to start one implementation of GenServer, it knows how to start _all implementations of GenServer_.

This property of interchangeability lends itself to use with the Dispatcher Pattern. Let's say that we are writing an application that takes the name of a product and tries to find a price for that item on an online marketplace. We could write our application like so:

```elixir
defmodule PriceFinder do
  @spec product_price(String.t()) :: integer
  def product_price(product_name) do
    cond do
      is_boat?(product_name) ->
        BoatTraderAPI.find_boat_price(product_name)
    	
      is_computer_component?(product_name) ->
        NeweggAPI.find_component_price(product_name)
    	
      is_car?(product_name) ->
        JoydriveAPI.find_car_price(product_name)
    end
  end

  # ...
```

This is fine, but consider the case where a developer makes a small change such that `BoatTraderAPI.find_boat_price/1` now returns a float instead of an integer. Whoops! Sadly, this is the kind of error that Dialyzer isn't likely to catch[¹]. To remedy this, a well-meaning developer might make the following change:

```diff
-      BoatTraderAPI.find_boat_price(product_name)
+      float_price = BoatTraderAPI.find_boat_price(product_name)
+      floor(float_price * 100)
```

However, now our `product_price/1` function has started to bloat with BoatTrader-specific logic.

So, what's the alternative? Let's write a behaviour and a dispatcher.

```elixir
defmodule PriceFinder do
  @callback product_price(String.t()) :: integer

  @spec product_price(String.t()) :: integer
  def product_price(product_name) do
    implementation(product_name).product_price(product_name)
  end

  defp implementation(product_name) do
    cond do
      is_boat?(product_name) -> BoatTraderAPI
      is_computer_component?(product_name) -> NeweggAPI
      is_car?(product_name) -> JoydriveAPI
    end
  end

  # ...
```

In this case, `BoatTraderAPI`, `NeweggAPI`, and `JoydriveAPI` (collectively, the implementations) should all contain `@behaviour PriceFinder`. This way, `PriceFinder.product_price/1` (the dispatcher) can be assured that a `product_price/1` function exists within each of those modules, and Dialyzer can check that the implementations conform to the typespec specified. Consequently, type-related errors should be harder to introduce and the dispatcher leaves no room for vendor-specific bloat to creep in.

## Protocols

Given what we now know about behaviours, the summary of protocols is somewhat confusing:

> A protocol specifies an API that should be defined by its implementations. A protocol is defined with `Kernel.defprotocol/2` and its implementations with `Kernel.defimpl/3`.

This sounds _exactly_ like what behaviours are for, and indeed, protocols use behaviours under-the-hood. However, where protocols and behaviours deviate is with a constraint that the summary did not specify. Namely, that each implementation of a protocol _must be for a distinct data type_.

To understand this further, let's take a look at a protocol in the Elixir standard library: `String.Chars`. While this may sound unfamiliar to you, you have certainly used it before as [`Kernel.to_string/1`](https://github.com/elixir-lang/elixir/blob/v1.12/lib/elixir/lib/kernel.ex#L3094-L3096):

```elixir
defmacro to_string(term) do
  quote(do: :"Elixir.String.Chars".to_string(unquote(term)))
end
```

All the macro is doing here is transforming your `to_string(my_string)` call to `String.Chars.to_string(my_string)` at compile-time.

Alright, so how then is `String.Chars.to_string/1` defined? This is the entirety of the `String.Chars` module with the documentation removed for brevity:

```elixir
defprotocol String.Chars do
  @spec to_string(t) :: String.t()
  def to_string(term)
end
```

However, just below that protocol's definition in [`string/chars.ex`](https://github.com/elixir-lang/elixir/blob/v1.12/lib/elixir/lib/string/chars.ex) are the implementations for many of Elixir's datatypes. Excerpted implementations for `Integer` and `Float`:

```elixir
defimpl String.Chars, for: Integer do
  def to_string(term) do
    Integer.to_string(term)
  end
end

defimpl String.Chars, for: Float do
  def to_string(term) do
    IO.iodata_to_binary(:io_lib_format.fwrite_g(term))
  end
end
```

As you can see, inside each `defimpl String.Chars, for: ...` there must be a `to_string/1` function defined that transforms its given datatype (here, either `Integer` or `Float`) to a string.

What's even more interesting, though, is that each Elixir struct is considered it's own "type" for the purposes of protocols. For fun, let's define our own type:

```elixir
defmodule Boat do
  defstruct [:length, :fuel_type]
end
```

Alright, and let's try to stringify an instance of this new struct:

```elixir
to_string(%Boat{length: 41, fuel_type: "diesel"})
#=> ** (Protocol.UndefinedError) protocol String.Chars not implemented for
#=>    %Boat{fuel_type: "diesel", length: 41} of type Boat (a struct)
```

If we want our `Boat` "type" to have a string representation, we'll have to implement the `String.Chars` protocol!

```elixir
defimpl String.Chars, for: Boat do
  def to_string(boat),
    do: "A #{boat.length}' boat powered by #{boat.fuel_type}"
end
to_string(%Boat{length: 41, fuel_type: "diesel"})
#=> "A 41' boat powered by diesel"
```

This has also exposed another power of protocols: a protocol can dispatch to a module that is not known to it. For instance, the authors of the `String.Chars` protocol had no idea that I would come along and implement their protocol for my `Boat` struct. And they didn't have to! Using protocols replaces the explicitly defined `implementation/1` function in the behaviour example above with something more flexible—so long as the implementation is determined solely based on the type of the argument.

The popular [Jason library](https://hexdocs.pm/jason) uses this concept with its `Jason.Encoder` protocol. You can implement the `Jason.Encoder` protocol to tell Jason how to JSON-encode your structs.

Alright, let's tie what we've now learned about protocols with our example from earlier, but with a slight twist. Instead of the user providing us the _name_ of a product, they will choose a product from our collection of `Boat`s, `Component`s, and `Car`s (where `Boat`, `Component`, and `Car` are each modules defining a struct).

Modifying our behaviour-based solution from earlier to fit this new model yields the following:
```elixir
defmodule PriceFinder do
  @callback product_price(any) :: integer

  @spec product_price(any) :: integer
  def product_price(product) do
    implementation(product).product_price(product)
  end

  defp implementation(product) do
    cond do
      is_struct(product, Boat) -> BoatTraderAPI
      is_struct(product, Component) -> NeweggAPI
      is_struct(product, Car) -> JoydriveAPI
    end
  end
end
```

Not bad! However, now that our `implementation` function is based solely around "types", we can use a protocol instead!

```elixir
defprotocol PriceFinder do
  @spec product_price(t) :: integer
  def product_price(product)
end

defimpl PriceFinder, for: Boat do
  def product_price(boat), do: BoatTraderAPI.product_price(boat)
end

defimpl PriceFinder, for: Component do
  def product_price(component), do: NeweggAPI.product_price(component)
end

defimpl PriceFinder, for: Car do
  def product_price(car), do: JoydriveAPI.product_price(car)
end
```

Whether you prefer the style of the behaviour approach or the protocol approach is a matter of personal taste. However, if our `PriceFinder` module is going to be shipped as part of a library, the protocol approach would allow users of the library to implement the `PriceFinder` protocol for their own structs.

## Conclusion

In short, both behaviours and protocols define an interface which must be fulfilled by its implementations. Behaviours are more general-purpose than protocols, and are sometimes used with dispatchers. Protocols are built on top of behaviours and have a type-based dispatcher built-in—one that is extensible to external types as well.
