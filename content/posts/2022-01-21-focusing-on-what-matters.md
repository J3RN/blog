---
layout: post
title: "Focusing on What Matters"
date: 2022-01-21
description: |
  It's easy to get lost when trying to evaluate a programming language and its ecosystem. Here, I propose a general framework for how developers should evaluate these based on the needs of the industry.
draft: true
tags:
- language design
- software development
- elixir
- ruby
- haskell
---

In one of my introductory computer science courses in college, my professor instructed us:

> Don't become attached to any IDE, text editor, programming language, or operating system. This field is young and constantly evolving; you don't want to be left behind.

I don't think I realized at that time what tremendously insightful advice this was. At that time I was primarily writing Java in Eclipse. Over the next few years I would work professionally in PHP and Ruby. I somewhat recently migrated to Elixir. I would soon move from Eclipse to Sublime Text and not long after replace that with Vim. After using Vim for about four years, I switched to Emacs and have been using that for the last five or so. My current development environment (Elixir, Emacs) is _fine_, but there's also ample room for improvement.

To her credit, I think my professor probably nailed the "why" of our situation. While developers are a fickle breed, our profession has only existed for roughly fifty years and most of the tools and languages that we use are younger than we are. If this doesn't strike you as incredible, consider that scalpels and trusses, implements used in modern medicine and bridge construction, were used by the Ancient Egyptians and Ancient Romans respectively.

However, I believe that software developers can make progress as a profession by putting aside the hype—the flavor-of-the-week frameworks, the fancy features and creative syntax—and focus on what matters.

What matters?
=================

The reason that we write software in the first place is create solutions that solve problems[^1]. Often times these problems can be solved without computers, and in many cases have been for years. Consider, for instance, that the calculations used to determine the trajectory of the Apollo missions were done by humans with pencils and paper. Computers, however, can solve some kinds of problems (like determining rocket trajectories) very quickly and therefore become a better solution to those problems.

Therefore, any software solution that doesn't solve the stated problem within given parameters is a failure. If I ask a computer to tell me what time it is and it tells me about the weather instead, this is impressive but also a complete failure as I still don't know what time it is. Similarly, if I ask for the time and only get it ten minutes later, chances are I'll have already gotten the time from somewhere else and the effort was wasted. The combination of problem and parameters under which the solution must be returned are generally referred to as "requirements". Given this, a language and its ecosystem (tooling, libraries, etc) should provide methods to verify that the software meets the requirements.

Requirements for considering a problem "solved" differ from problem to problem. What follows are the typical kinds of requirements that I see in my day-to-day work. These are in no particular order.

### Logical Correctness

Being "logically correct" means producing the desired output; solving the problem. For instance, if I type `1 + 1` into a calculator application and it replies with `5`, this is a logical error, and the application is not logically correct. Similarly, if I type some information into a form and click the "Save" button, this information should be persisted for later retrieval or else this is also a logical error.

The most common and intuitive methodology that languages and their ecosystems provide to verify the logical correctness of applications is automated testing. All modern languages that I'm aware of have libraries for writing and tooling to run automated tests, but this problem is not yet solved. Software developers are embroiled in wide-ranging religious wars of _what_ should be tested and _how_ those tests should function.

In my mind, the answer to these questions are clear: A test should impersonate a user of the system and verify the expected output as a user would. If your application is used by human beings with their web browsers, then your tests should open a web browser and click, scroll, and type as a human would. If your application is a software library, then your tests should emulate an application that pulls yours in as a dependency and verify that it does what it purports to do. When designing your tests, ask "Who are my users?", "How will they interact with my application?", and "What do they expect to happen?".

A programming language and/or its ecosystem should provide tooling and libraries for ensuring logical correctness as described here.

There is also another way in which programming languages can aid developers in writing logically correct programs: encouraging clarity[^2]. This topic is deep enough to warrant its own blog post (one is planned! 😀), but at a high level what I mean by clarity is "the ability for another developer to understand a piece of code". One aspect of this is that a developer should be able to understand how an application works such that they have a high degree of confidence that it is logically correct. As has been pointed out a multitude of times before, code has two readers: the machine and other developers. Both are important.

I believe that a language's tooling can _and should_ guide developers towards writing more comprehensible code, and this is an area that I am still actively exploring.

### Productivity

If you've ever worked as a professional software developer, you'll know that there's always someone wanting their application completed as fast as possible. Completing functionality quickly builds confidence and goodwill with your clients and can be a strategic differentiator for your company. Rather than work late and forego relationships with family and friends, we (software developers) should prefer if creating, extending, and modifying software systems were quick and effortless.

Unfortunately, the word "productivity" also lacks a widely-accepted definition in the software industry, so for the purposes of this article "productivity" refers to the amount of time and effort required to extend or modify a system[^4]. Less time and effort is "more productive", more time and effort is "less productive".

The language, framework, libraries, and architectures of an application largely determine the productivity of software engineers working on the system.

### Stability

The term "stability" here means that the application does not crash.

### Availability

The application should be accessible when it is needed. A crash in one part of the system should not affect the rest of the system.

### Safety

It seems that every day a company's software systems are breached and the data that users had entrusted to them is leaked. This is an example of a system that is _unsafe_; a system that is "safe" does not harm its users and the users are harmed by their personal information being released. There are a number of ways that users can be harmed by software; physically (think robotics), emotionally (think Facebook), 

### Speed

[^1]: Software is also sometimes written for fun—e.g. the TIS-100 video game or solving contrived programming puzzles—but that is not the focus of this article.
[^2]: The first section of [this Saša Jurić talk by the same name] refers to what I am talking about here.
[^4]: There is also no consensus on how to measure the productivity of software developers.

[this Saša Jurić talk by the same name]: https://www.youtube.com/watch?v=6sNmJtoKDCo
[ISWIM]: https://www.cs.cmu.edu/~crary/819-f09/Landin66.pdf
