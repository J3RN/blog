---
layout: post
title: "My API Integration Methodology"
date: 2025-07-27
description: A general three-part methodology for building robust API integrations.
tags:
- elixir
- APIs
- design
---

The code examples in this post are in Elixir, and some of the design aspects are Elixir-specific, but the concepts are translatable to most languages.

A common aspect of modern web applications is the need to communicate with other web applications in what is usually referred to as "API integrations".  For example, consider [Stripe].  Stripe is a payment processor service that handles the complicated parts of accepting payments.  If your app is being extended to have a checkout experience for customers, you might integrate with Stripe to accomplish that.

That said, the "how" of implementing API integrations is more of an art than a science.  Typically, each team or developer has their own style for integrations, and what follows is my approach, which is largely based on the idea of bounded contexts from Chapter 14 of [Domain-Driven Design].

My typical design for an API integration consists of the following parts:
1. Defining how such a service will be utilized in your application's domain.
2. Wrapping the service API, in its domain.
3. Tying the two together with an adapter module.

## Your Application's Domain

A common mistake that developers make when integrating a vendor is to couple their application to the details of the vendor.  For instance, one approach for integrating Stripe into an application—let's call it `Bookstore`—would be to either create or install a Stripe library that contains all necessary code for communicating with Stripe, providing functions such as `Stripe.purchase_item`.  Once added to the project, developers could add calls to `Stripe.purchase_item(item)` throughout the application.  While this is a very concise approach, the developers would run into issues should the company want to switch to a new payment processor, such as [PayPal].  In this case, the developers could attempt to replace all calls to `Stripe.purchase_item` with some equivalent function in a PayPal library and hope that the types expected by and returned from `purchase_item`, etc., were not Stripe-specific or at least similar enough to the PayPal library's types to not cause too much trouble.  Better yet, there's the possibility that the company wants to support _both_ Stripe and PayPal, and will allow customers to choose which service they'd like to use.  In this scenario, the developers either need to find all the places where they called `Stripe.purchase_item` and update them to be able to determine which payment processor to use, or alternatively, create some kind of abstraction over the `Stripe` and new `PayPal` libraries, perhaps `Bookstore.PaymentProcessor`, which knows how to delegate to the appropriate service.

The latter approach is the one that I typically start with.  Specifically, the abstraction module—here `Bookstore.PaymentProcessor`—serves two purposes:

1. It defines an interface that the rest of our application code should call for its payment processing needs.  Instead of calling vendor modules directly, our application code should call the abstraction, which will then dispatch to the appropriate vendor.
2. It defines an interface that adapter modules (Section 3) should implement as a [behaviour].

These two interfaces can be, but do not need to be, the same.  For instance, if *all* of our vendors split an action in our application's domain (e.g., "place an order") into two sequential actions (e.g., "create invoice" and "finalize invoice"), it might be fortuitous for the application-facing interface to have a single function (e.g., `place_order`) and the interface that the adapter modules implement to have two functions (e.g., `create_invoice`, `finalize_invoice`).  These interfaces being different is not typical, but can occasionally be advantageous.

For the this example application's payment processor abstraction, let's start with the behaviour:
```
defmodule Bookstore.PaymentProcessor.Behaviour do
  @callback purchase_books(Bookstore.Customer.t(), [Bookstore.Book.t()]) ::
              {:ok, Bookstore.PaymentConfirmation.t()} | error()

  # ... other functions that our application requires of payment processors
end
```

Next, because the two interfaces for this example are the same, the `Bookstore.PaymentProcessor` will implement this behaviour by dispatching to vendor adapter modules (Section 3) that also implement it.

```elixir
defmodule Bookstore.PaymentProcessor do
  @behaviour Bookstore.PaymentProcessor.Behaviour
  
  @impl Bookstore.PaymentProcessor.Behaviour
  def purchase_books(customer, books) do
    vendor_adapter().purchase_books(customer, books)
  end

  # ... other dispatches like the above for other required functions

  defp vendor_adapter do
    # ... logic for choosing which vendor should be used
  end
end
```

Note that the types consumed and returned by the `Bookstore.PaymentProcessor` module are *Bookstore* types, not Stripe or PayPal types.  The rest of the application is shielded from all of the peculiarities of particular vendors by our `Bookstore.PaymentProcessor` module.  More on that later.

In this example, the `vendor_adapter` function chooses which vendor adapter module we want to handle this request.  If the application only uses a single vendor at a time, we could define the vendor adapter in our config:

```
config :bookstore, payment_adapter: Bookstore.PaymentProcessor.StripeAdapter
```

And fetch it like so:

```
defp vendor_adapter do
  Application.compile_env!(:bookstore, :payment_adapter)
end
```

This approach can also be used to set a dummy payment adapter for the test suite so that the application isn't calling out to Stripe every time it runs (which has its own trade-offs; a topic for a different post).

Alternatively, the `vendor_adapter` function might consume some data, say the `customer` struct, and determine the vendor adapter to use from that (e.g., the customer's configured preference).  So long as `vendor_adapter` returns module that implements the behaviour, you can put any logic you like in it.

## The Vendor's Domain

Just as your application has models like `Customer`, `Order`, etc., and relationships between them, your vendors' applications do as well.  Furthermore, a vendor's concept of, say, `Customer`, and its relationships to other models in their domain, are likely not exactly the same, or even very similar, to the `Customer` model and its relationships in your application.  Communicating with a vendor means speaking to them in terms of their own models and relationships rather than those of your own application.

If you're lucky, someone has written a library that already does this.  For many popular vendors, companies like Stripe and PayPal, there are almost certainly libraries available from your package manager that wrap requests to the vendor in terms of its own domain.  If need be, you can write your own wrapper around a vendor's API[^1].  For your own sanity, don't try to skip steps and create vendor-specific functions that operate in terms of your application's domain; you'll wind up with scenarios where a term like `invoice` has two distinct meanings interwoven within a single function.  Additionally, by sticking to the vendor's domain in this API wrapper, you give yourself the opportunity to share this code between projects; even with the whole world by publishing it to a package repository.

## The Adapter Module

As we've established: there are two domains, that of your application and that of your vendor.  Models, relationships, and actions in your application's domain generally correspond to models, relationships, and actions in your vendor's domain (which is spoken by the vendor module/library from the previous section), but these correspondences are not always 1:1.  For instance, your application's `Order` model might correspond to both the `Order` and `Invoice` model in your vendor's domain.  Sometimes a single action in your application's domain (e.g., "place an order") corresponds to a series of actions within your vendor's domain (e.g., "create invoice" and "finalize invoice").

An adapter module provides the mapping between these two.  Its interface speaks in your application's domain, using the behaviour defined in Section 1, and its implementation utilizes models and actions of the vendor's domain.

For instance:

```
defmodule Bookstore.PaymentProcessor.StripeAdapter do
  @behaviour Bookstore.PaymentProcessor.Behaviour
  
  @impl Bookstore.PaymentProcessor.Behaviour
  def purchase_books(customer, books) do
    stripe_customer = to_stripe_customer(customer)
    stripe_line_items = to_stripe_line_items(books)
    stripe_invoice = create_stripe_invoice(stripe_customer, stripe_line_items)
    
    with {:ok, stripe_invoice_id} <- Stripe.create_invoice(stripe_invoice),
         {:ok, final_stripe_invoice} <- Stripe.finalize_invoice(stripe_invoice_id) do
      {:ok, to_payment_confirmation(final_stripe_invoice)}
    else
      error -> translate_error(error)
    end
  end

  # ... various mapping implementations
end
```

At the beginning of this implementation, models in the application's domain are mapped to those in the vendor's domain.  Additionally, once in the vendor's domain, we need to combine the multiple models into one.  Next, the adapter module performs the sequence of actions in the vendor's domain that map to the single "purchase" action in the application's domain.  Lastly, results from the vendor's domain are translated to the application's domain.

## Conclusion

Using this methodology, most of the application does not need to know the details of particular vendors.  Instead, Bookstore application developers can create calls to `Bookstore.PaymentProcessor` with the models they're already using and receive back a model in the application's domain.  By abstracting over the details of a vendor, it is easier for developers to replace a vendor or add a second vendor for the same service.

One word of caution here: don't go overboard.  You *can* use this method to encapsulate *everything* your application doesn't own, like HTTP clients and JSON parsers.  However, doing so would introduce *a lot* of code and thus complexity into your codebase.  My advice is to only use this approach for **vendors that are liable to change**.  If your business is pretty certain that they will only ever use Stripe and never want to switch vendors or add another (which sounds pretty worrisome to me), then you can feel confident just calling `Stripe.purchase_item` directly and skip writing three or four additional modules.  But if the future of an integration is unclear, I believe this is the approach you should take.

[^1]: Frankly, writing your own wrapper for a vendor API, even if a library already exists, can be fortuitous.  The existing library might stop being maintained or, in rare scenarios, become malicious; risks avoided by maintaining your own wrapper.

[Stripe]: https://stripe.com
[PayPal]: https://www.paypal.com
[Domain-Driven Design]: https://www.domainlanguage.com/ddd/blue-book/
[behaviour]: https://elixir-lang.org/getting-started/typespecs-and-behaviours.html#behaviours
