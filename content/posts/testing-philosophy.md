---
title: "Testing Philosophy"
date: 2022-05-13
layout: post
draft: true
---

The most common and intuitive methodology that languages and their ecosystems provide to verify the logical correctness of applications is automated testing. All modern languages that I'm aware of have libraries for writing and tooling to run automated tests, but this problem is not yet solved. Software developers are embroiled in wide-ranging religious wars of _what_ should be tested and _how_ those tests should function.

In my mind, the answer to these questions are clear: A test should impersonate a user of the system and verify the expected output as a user would. If your application is used by human beings with their web browsers, then your tests should open a web browser and click, scroll, and type as a human would. If your application is a software library, then your tests should emulate an application that pulls yours in as a dependency and verify that it does what it purports to do. When designing your tests, ask "Who are my users?", "How will they interact with my application?", and "What do they expect to happen?".

---

We were testing the same functionality at various layers of abstraction.  This is annoying because you wind up with a bunch of duplicate tests and when extending the system you feel obligated to write duplicate tests for consistency's sake.  That, in turn, is problematic (not just because it takes additional time, but) because when the functionality being tested changes, you then have to go through the two suites of tests at their different levels of abstraction and update them all, which takes time and effort.

In general, tests ossify.  Once a feature is merged, tests have preventing change as their primary purpose.  When you're developing a feature, tests can show that the feature works.  But once the feature is accepted and merged in, the tests' new job is to ensure that the feature keeps working; they become de-facto regression tests.  As a side-effect, intentionally changing code that has tests is harder because you need to expend time and effort updating the tests.  e.g. If you write tests against a function named `do_thing` and that function is updated to take four arguments instead of three, you must go through each test for `do_thing` and update the arguments being passed in the test.  It may take a while, depending on what you're adding!

Given that, when writing tests I ask myself "What do I want to prevent from changing?" and also "What do I **not** want to prevent from changing?".  To me, for Instinct, the answer to the first question is generally that I want the API that the front-end consumes to be consistent.  That's pretty important, because if the API changes unexpectedly, the front-end could break and cause issues for our users.  Tests at the context level, however, have the issue of ossifying details that I actually don't really care about at all; like what module `update_account` is defined in or how many arguments that function takes.  In my mind, if the API still works, then these details are totally irrelevant!  If someone wanted to rename `Instinct.Owner` to `Instinct.Account` or something, they should be able to complete that work without updating my API tests at all.
