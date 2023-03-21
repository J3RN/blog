---
layout: post
title: "Focusing on What Matters"
date: 2022-04-01
description: |
  It's easy to get lost when trying to evaluate a programming language and its ecosystem. In this post, I propose a set of qualities that developers should look.
tags:
- language design
- software development
---

In an introductory computer science course, our professor instructed us:

> Don't become attached to any IDE, text editor, programming language, or operating system. This field is young and constantly evolving; you don't want to be left behind.

I don't think I realized at that time what tremendously insightful advice this was.  At that time I was primarily writing Java in Eclipse.  Over the next few years I would work professionally in PHP and Ruby and then migrate to Elixir.  I would move from Eclipse to Sublime Text to Vim, and after using Vim for about four years, switch to Emacs.  My current development environment (Elixir, Emacs) is _fine_, but there's ample room for improvement.

To her credit, I think my professor probably nailed the "why" of our situation.  While developers are a fickle breed, our profession has only existed for roughly seventy years and most of the tools and languages that we use are younger than we are.  If this doesn't strike you as incredible, consider that scalpels and trusses, implements used in modern medicine and bridge construction, were used by the Ancient Egyptians and Ancient Romans respectively.

However, I believe that software developers can make progress as a profession by putting aside the hype—the flavor-of-the-week frameworks, the fancy features and creative syntax—and focusing on what matters.

What matters?
=================

The reason that we write software in the first place is to create solutions to problems[^1].  Therefore, any software that doesn't solve the stated problem within given parameters is a failure. If I were to ask a computer to tell me the trajectory needed to get a Saturn V rocket to the moon and it tells me the weather instead, I would be impressed but it would also be a complete failure as I still don't know the answer to my question.  Similarly, if the computer only produces the correct trajectory after churning for a decade, chances are I'll have already figured it out by other methods and the effort was wasted.  This combination of problem and parameters are generally referred to as "requirements".

![Saturn V rocket](/images/saturn_v.webp)
<small>Photo by <a href="https://unsplash.com/@nasa?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">NASA</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

Software developers should reach for tools (programming languages, software libraries, etc) that help them create software that meets its requirements.

Requirements differ, but what follows are the typical kinds of requirements that I see in my day-to-day work.  These are in no particular order.

### Logical Correctness

Being "logically correct" means producing the desired output; solving the problem. For instance, if I type `2 + 2` into a calculator application and it replies with `5`, this is a "logical error", and the application is not logically correct. Similarly, if I type some information into a form and click the "Save" button, this information should be persisted for later retrieval or else this, too, is a logical error.

![a whiteboard displaying "2 + 2 = 5"](/images/math.jpg)

<small>Photo by <a href="https://unsplash.com/@michalmatlon?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Michal Marlon</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Productivity

While the word "productivity" lacks a widely-accepted definition in the software industry, for my purposes "productivity" refers to the amount of time and effort required to create, extend, or modify a system.  Less time and effort is "more productive", more time and effort is "less productive".

If you've ever worked as a professional software developer, you'll know that stakeholders always want their application completed as fast as possible.  Completing functionality quickly builds confidence and goodwill with your stakeholders and can be a strategic differentiator for your company.  Rather than working extra hours and foregoing relationships with family and friends, software developers should reach for tools that make creating, extending, and modifying software systems quick and effortless.

![a kanban board](/images/kanban.jpg)
<small>Photo by <a href="https://unsplash.com/@lazizli?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Lala Azizli</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Stability

The term "stability" here means that the application does not crash.  If you view a crash as "incorrect output" (e.g. a server produced a 500 response instead of the expected 200 response), one might view stability as saying "an application should be logically correct _all the time_ and not just sometimes."

![a Jenga tower](/images/jenga.jpg)

<small>Photo by <a href="https://unsplash.com/@oscaresquivel?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Oscar Ivan Esquivel Arteaga</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Availability

An application should be accessible when it is needed.  If an application is only needed twice per day and is available at those times and is unavailable at all other times, it is still meeting its availability requirement (e.g. a timesheet application may have such a requirement).  However, if a user tries to use a piece of software and cannot, it is not meeting its availability requirement.

![a web browser displaying an error page](/images/404.jpg)

<small>Photo by <a href="https://unsplash.com/@introspectivedsgn?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Erik Mclean</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Safety

It seems that every day a company's software systems are breached and the data that users had entrusted to them is leaked. This is an example of a system that is _unsafe_, whereas a system that is "safe" does not harm its users. There are a number of ways that users can be harmed by software; physically (think robotics), emotionally (think Facebook), financially (think MtGox), and others.

![a guard rail](/images/railing.jpg)

<small>Photo by <a href="https://unsplash.com/@introspectivedsgn?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Erik Mclean</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Speed

Sometimes speed is a "soft" requirement.  While users would like pages to load instantaneously in their browsers, they generally will still be satisfied so long as it loads in a second or so[^2].  In other cases, such as audio processing or transmission, a delay of even 100ms is totally untenable as the user would detect that the audio was distorted.  In some extreme cases, the software speed requirement is specified down to the microsecond such that the system can interoperate with another system operating at a high, set frequency.  Whatever the number is, software only meets its speed requirement if it is sufficiently fast.

![a stopwatch](/images/stopwatch.jpg)
<small>Photo by <a href="https://unsplash.com/@tsvetoslav?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Tsvetoslav Hristov</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>


Going Forward
=============

This list is not necessarily comprehensive, and I reserve the right to amend it in the future.  In any event, though, I will try to tie future posts around languages and their ecosystems back to this post to show how the topic is related to a concept that _matters_.

[^1]: Software is also sometimes written for fun—e.g. the TIS-100 video game or solving contrived programming puzzles—but that is not the focus of this article.
[^2]: Unsurprisingly, "how fast should a page load?" features a variety of answers from "700ms" to "under 3s". [Google's PageSpeed Insights states that a "good" First Contentful Paint (FCP) time is less than 1800ms](https://developers.google.com/speed/docs/insights/v5/about)
