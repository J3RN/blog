---
layout: post
title: "How I Design API Integrations"
date: 2021-12-05T15:34:07-05:00
draft: true
description: |
  A general three-part methodology for building robust API integrations.
tags:
- elixir
- APIs
- design
---

A common aspect of modern web applications is the need to communicate to other web applications. Such communication is generally termed an "API integration," and typically makes use of a documented interface (the "I" in "API").

For example, consider [Stripe]. Stripe is a payment processor service that handles the complicated parts of accepting payments. If your app wants to have a checkout experience for your customers, you might integrate with Stripe to accomplish that.

That said, the "how" of implementing API integrations is more of an art than a science. Typically each team or developer has their own style for integrations, and what follows is my approach which is largely based on the idea of bounded contexts from Chapter 14 of [the DDD book].

My typical design for an API integration consists of the following parts:
1. Defining how the such a service will be utilized, in your application's domain.
2. Wrapping the service API, in its domain.
3. Tying the two together with an integration module.

## Preparing Your Application's Domain

A common mistake that developers make when integrating a vendor is to couple their application to the details of the vendor.  For instance, one such approach for integrating Stripe into an application—let's call it `Bookstore`—would be to create a new `Bookstore.Stripe` module which contains all necessary code for communicating to Stripe, providing functions such as `purchase_item`.  Then, throughout the application, we would have calls like `Bookstore.Stripe.purchase_item(item)`.  While this is a very concise approach, we would run into issues should our company want to switch to a new payment processor, such as [PayPal].  Sure, then we could rename `Bookstore.Stripe` to `Bookstore.PayPal`, rewrite its internal implementation, update the references throughout our codebase, and hope that the types expected by and returned from `purchase_item`, etc, were not Stripe-specific (we're in a whole mess of trouble if they were).  Better yet, there's the possibility that our company wants to support _both_ Stripe and PayPal, and will allow the customer to choose which service they'd like to use.  In this scenario, we'd either need to find all the places where we called `Bookstore.Stripe.purchase_item` and update them to be able to determine which payment processor to use.  Alternatively, could create some kind of abstraction over our existing `Bookstore.Stripe` and new `Bookstore.PayPal` modules, perhaps `Bookstore.PaymentProcessor`, which knows how to delegate to the appropriate service.

The latter approach is the one that I typically start with. Specifically, the abstraction module—here `Bookstore.PaymentProcessor`—serves two purposes:

1. It defines an interface that the rest of our application code should call for its payment processing needs. Instead of calling vendor modules directly, our application code should call the abstraction which will then dispatch to the appropriate vendor.
2. It defines an interface that vendor modules should implement (Section 3) as a [behaviour].

These two interfaces can be, but do not need to be, the same.

For our payment processor abstraction, part of our `Bookstore.PaymentProcessor` module would look like this:

```elixir
defmodule Bookstore.PaymentProcessor do
  @callback purchase_item(Bookstore.Customer.t(), Bookstore.Book.t()) ::
              Bookstore.PaymentConfirmation.t()

  # ... other parts of the behaviour required of vendor modules
  
  @spec purchase_item(Bookstore.Customer.t(), Bookstore.Book.t()) ::
          Bookstore.PaymentConfirmation.t()
  def puchase_item(customer, book) do
    vendor().purchase_item(customer, book)
  end

  # ... other parts of the interface for our application code to call

  defp vendor do
    # ... logic for choosing which vendor should be used
  end
end
```

[Stripe]: https://stripe.com
[PayPal]: https://www.paypal.com
[the DDD book]: https://www.domainlanguage.com/ddd/blue-book/
[behaviour]: https://elixir-lang.org/getting-started/typespecs-and-behaviours.html#behaviours
