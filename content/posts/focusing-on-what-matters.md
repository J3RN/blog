---
layout: post
title: "Focusing on What Matters"
date: 2022-04-01
date_updated: 2025-07-12
description: |
  It's easy to get lost when trying to evaluate a programming language and its ecosystem. In this post, I propose a set of qualities that developers should look.
tags:
- language design
- software development
aliases:
- /posts/2022-04-01-focusing-on-what-matters
---

Update 2025-07-12: I merged two sections, Stability and Availability, into one section, Usability, that also encompasses topics such as accessibility.  I also updated some paragraph structures and removed the intro because it was weak.

What matters?
=================

The reason that we write software in the first place is to create solutions to problems[^1].  Around the world, every day people face problems that can be solved by computers.  A person might want to know "What are the chances it will rain tomorrow?"  A statistical model can provide a reasonable estimate.  A person might want to know "If I have a loan at 3%, how much will I pay in interest over the term of the loan?"  Accounting software can answer this question.

However, software that doesn't solve the problem within certain parameters is a failed solution. If I were to ask a computer to tell me the trajectory needed to get a Saturn V rocket to the moon and it tells me the weather instead, I still wouldn't know the answer to my question.  Similarly, if the computer only produces the correct trajectory after churning for a decade, chances are I'd have already figured it out by another method and the effort was wasted.

This combination of problem and parameters are generally referred to as "requirements".

![Saturn V rocket](/images/saturn_v.webp)
<small>Photo by <a href="https://unsplash.com/@nasa?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">NASA</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

Software developers should reach for tools (programming languages, software libraries, etc) that help them create software that meets its requirements.

Requirements differ, but what follows are the typical kinds of requirements that I see in my day-to-day work.  These are in no particular order.

### Correctness

Being "correct" means producing the desired output; solving the problem. For instance, if I type `2 + 2` into a calculator application and it replies with `5`, this is what we would call "incorrect". Similarly, if I type some information into a form and click the "Save" button, this information should be persisted for later retrieval or else this, too, is incorrect behavior.

![a whiteboard displaying "2 + 2 = 5"](/images/math.jpg)

<small>Photo by <a href="https://unsplash.com/@michalmatlon?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Michal Marlon</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Productivity

While the word "productivity" lacks a widely-accepted definition in the software industry, for my purposes "productivity" refers to the amount of time and effort required to create, extend, or modify a system.  Less time and effort is "more productive", more time and effort is "less productive".

If you've ever worked as a professional software developer, you'll know that stakeholders always want their features completed as fast as possible.  Completing functionality quickly builds confidence and goodwill with your stakeholders and can be a strategic differentiator for your company.  Rather than working extra hours and foregoing relationships with family and friends, software developers should reach for tools that make creating, extending, and modifying software systems quick and effortless.

![a kanban board](/images/kanban.jpg)
<small>Photo by <a href="https://unsplash.com/@lazizli?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Lala Azizli</a> on <a href="https://unsplash.com?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a></small>

### Usability

The term "usability" here means simply that users are able to use the software.  Usability exists on scale.  For instance, if your software requires vision to operate, it is unusable by blind users.  If your software requires knowledge of technical concepts in order to operate, it is "somewhat unusable" (as a particularly motivated individual lacking such knowledge could expend the time and effort to gain the technical knowledge required, but many potential users cannot).

Another example of "unusable software" is an application that is offline (i.e. the website is down) or crashed (e.g. suddenly closes, refuses to open, etc).

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
