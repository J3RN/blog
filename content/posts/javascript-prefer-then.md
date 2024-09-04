---
layout: post
title: "Why I Still Use JavaScript's then"
date: 2024-09-04
description: |
  JavaScript has a readable way to resolve Promises with `await`.  Why do I still use `then` instead?
tags:
  - javascript
---

With ECMAScript 2017, JavaScript gained the [`async`] and [`await`] keywords as a new way to deal with [Promise]s. The traditional way to work with Promises was with the methods [`then`], [`catch`], and [`finally`], like so:

```javascript
const fetchResource = () => {
  return fetch("https://example.com/api/resource")
    .then((res) => handleResponse(res))
    .catch((err) => handleError(err))
    .finally(() => cleanup());
};
```

The above function:

- fetches a data from a server using the venerable [`fetch`] function, which returns a Promise,
- calls the `then` method on the Promise returned from `fetch` to configure it such that if the Promise "resolves" (completes successfully), the `handleResponse` function is invoked with the "resolved" value (here, an HTTP response),
- calls the `catch` method on the Promise returned from `then` which configures it such that if the Promise "rejects" (is unsuccessful), the `handleError` is invoked with the reject value (some kind of error),
- calls the `finally` method on the Promise returned from `catch` which configures it such that the `cleanup` function is always invoked regardless of whether the Promise resolves or rejects.

Technically, the `catch` and `finally` method are wrappers around `then` that specialize it for different scenarios and allow programmers to convey intent more easily.

This pattern has a clear analogue to monads in other languages, with `then` being analogous to Haskell's `>>` operator (named "then") and `>>=` operator (named "bind"). But then again, Haskell programmers often prefer the more concise `do` notation to `>>` and `>>=`. JavaScript's version of a clearer notation is `async` and `await`; applied to the example above, this yields:

```javascript
const fetchResource = async () => {
  try {
    const res = await fetch("https://example.com/api/resource");
    handleResponse(res);
  } catch (err) {
    handleError(err);
  } finally {
    cleanup();
  }
};
```

In order to convert from the first example to the second, the anonymous function being bound to `fetchResource` is declared as `async` so that `await` can be used inside of it. An `async` function, when invoked, always returns a Promise; something that was optional and explicit in the first example (courtesy of the `return` keyword), is implicit and mandatory in the second.

Next, we update our `fetch(...)` call to have the `await` keyword in front of it. Semantically, this implies that execution of this thread will pause while we wait for the remote server to reply, "awaiting" its response (the actual implementation is somewhat more nuanced). The value of `await fetch(...)` is the "resolved" value of the `fetch`, here an HTTP response, which we assign to the `res` variable.

But what if `fetch` rejects? In that case, `await fetch(...)` would throw the reject value (some kind of error) and therefore need to be caught if we want to handle that situation. Therefore, we wrap our `await fetch(...)` call in a `try...catch...finally` so that we can catch that thrown reject value and handle it (and `finally` is here for cleanup, as before).

While this second version is more lines of code, it has a somewhat more familiar linear style and forgoes the need to introduce anonymous functions throughout. It also uses the `try...catch` construct, which is familiar to programmers of popular languages such as Java and C#.

So... what's the catch? (Pun only somewhat intended)

The _catch_ is that the `catch` branch of the `try...catch` statement catches **all** thrown errors, not just Promise rejections.

To make this concrete, imagine that there's a bug in our `handleResponse` implementation, and it encounters a `TypeError` (e.g. `foo is undefined`). This `TypeError` thrown by `handleResponse` will be caught by the enveloping `try...catch` construct, and therefore passed to our `handleError` function. This isn't necessarily a _bad thing_ if we anticipated this outcome, but it is a significant departure from the `then`-based example, where such an error would not be caught by the `catch` method's effect on the Promise. If you write your code like I do, you'd have only anticipated that `handleError` was going to be invoked with Promise rejection values (from `fetch`), and this `TypeError` from `handleResponse` winding up in `handleError` is going to cause further issues.

Of course, we could attempt to simulate the `then` behavior by verifying that `err` is the sort of data we expected:

```javascript
const fetchResource = async () => {
  try {
    const res = await fetch("https://example.com/api/resource");
    handleResponse(res);
  } catch (err) {
    if (err.message.startsWith("NetworkError")) handleError(err);
    else throw err;
  } finally {
    cleanup();
  }
};
```

Now, you might be wondering why I'm checking `err`'s message instead of using `instanceof`. This is due to a terrible truth that I just learned: For whatever reason, `fetch`'s network errors are instances of `TypeError`, and if I had written `err instanceof TypeError`, the bug would be no more fixed than before.

Hopefully by now you understand my rationale for preferring `then` chaining over `await`.

Am I missing something? Did I get it all wrong? [Send me a message on Mastodon] and let me know!

[`async`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/async_function
[`await`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await
[Promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
[`then`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then
[`catch`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch
[`finally`]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally
[`fetch`]: https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch
[Send me a message on Mastodon]: https://fosstodon.org/@j3rn
