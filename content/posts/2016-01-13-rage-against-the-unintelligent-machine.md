---
layout: post
title: "Rage Against the Unintelligent Machine"
description: |
  "Classical AI" (fancy single-purpose algorithms) is still being taught in university AI courses. In this post I argue that computer science, as a field, should focus more on general AI and neuromorphic computing, while also noting ambiguity of the word "intelligence."
date: 2016-01-13 10:08:00 -0500
tags:
- ai
- machine learning
---

Today marked the first time I was ever so uncomfortable in a class that I nearly got up and walked out. Such a level of discomfort is not so unfamiliar, though. It was, in fact, common during my review of candidates' projects during my involvement in one of my former employer's hiring process. Someone might ask the candidate something like "How does user authorization work?" They'd drone on about encryption, throwing out buzzwords and waiting to see a spark in someone's eye, letting them know that they weren't a complete failure. It would never come. I would start getting jittery. I would feel an irrepressible urge to jump up and yell profanity, to tear out my hair and flip things.

It was like that, but in a university course. The professor started at the natural beginning, "What is A.I.?" A natural question that you would rather hope students would know the answer to as a prerequisite for taking the course. "What," you might ask, "*is* his definition of A.I.?" A machine that need only behave intelligently.

> A machine that need only behave intelligently

At this point, I shuffled uncomfortably in my chair. While this concept dates back to Dr. Alan Turing, a man widely considered the father of modern computing, this idea of a facade of intelligence with no real intelligent underpinnings disturbs me. Dr. Turing, as you may know, is widely known for creating the "Turing Test," a common standard of intelligent behavior among A.I. researchers. To pass the Turing Test, a machine need only fool a human interrogator into believing that it is a human. Several programs, even rather trivial ones, have been able to pass this test from time to time. I do not believe these programs deserve the badge of "intelligence," artificial or not.

> Several programs, even rather trivial ones, have been able to pass this test from time to time

Let's say you disagree. Let's say that you think that there is no need to understand how a human brain (or otherwise) works in order to replicate its output. Fine, let's agree to disagree and move on. So you want to create a machine that will act "intelligent." What is this "intelligence" anyway? Just on cue, the professor ever so slightly bumps the issue. He states something akin to:

> "Intelligence is hard to define. For that reason, we don't. We just know intelligence when we see it. We can interact with a thing, and know that it is intelligent."

I recoil in my seat. You think that you just *know* intelligence? You don't think you *might* want to have a destination in mind before starting to run full speed? It's really no wonder the A.I. field doesn't seem to have progressed since the 1950's! Why is it that we humans expect ourselves to be the arbiters of intelligence? Are we able to distinguish the intelligent from the unintelligent through some sixth sense? You may have already deduced this expectation of a human being able to judge intelligence from the Turing Test itself. As it turns out, not all humans are so intelligent. I had rather hoped that someone might have wondered "If it must seem intelligent, shouldn't we try to actually define what that means?" Evidently not.

Besides, what gave anyone the idea that being able to discern the intelligent from the unintelligent was a sufficient condition to be able to create A.I.? I can tell you if an object is or is not a car, but that doesn't qualify me to actually build one.

> Why is it that we humans expect ourselves to be the arbiters of intelligence?

But wait! Here, the professor brings up a definition of intelligence from John McCarthy, a long-time A.I. researcher and the inventor of LISP. The quote is as follows:

> "Intelligence is the computational part of the ability to achieve goals in the world."

I think on this for a minute. My intuition tells me that this is utter rubbish, but I'm not going to just blow off a quote by a major computer scientist on my intuition alone. Let's break this quote down.

> "...the ability to achieve goals in the real world"

OK, sure. There's some goal that needs achieving. Like writing this blog post, catching a ball, holding a conversation, etc. How does intelligence fit in again?

> "Intelligence is the computational part of..."

I considered this thoughtfully. Merriam-Webster defines "computational" in terms of "compute", which it defines, in turn, as "to find out (something) by using mathematical processes." What part of writing this blog post requires mathematical processes? What part of catching a ball or holding a conversation involves mathematical processes? It is true that you could write an algorithm to transform my notes from class (input) to this blog post (output), much like in Searle's [Chinese Room](http://plato.stanford.edu/entries/chinese-room/). But is that really "acting intelligent?" Would such an algorithm be able to create different blog posts for different inputs as a human would be able to? What if an entire life (a larger context) was fed as input as well?

> But is that really "acting intelligent?"

At an abstract level, it seems somewhat believable. I believe that you could, perhaps, abstract the workings of the brain into a mathematical model, though it would be rather complex. I'm not inclined, however, to believe that Dr. McCarthy and I agree on a level of abstraction. Besides, A.I. researchers don't care about real intelligence, and thus brains, anyway, remember?

Suddenly, things took a turn for the worse. The professor gave forth the reasoning for "Weak A.I." (just trying to emulate output) over "Strong A.I." (simulating a brain, or brain-like functions): Brains are complex. Frankly, this goes without saying. If brains were simple, there would be some sort of concensus over how they work at a variety of levels, which there certainly is not.

> Brains are complex

However, as an excuse for not attempting to understand or replicate the inner workings of brains in the pursuit of intelligent behavior, this idea is ludicrous. To imagine that replicating each of millions of features of a brain, separately, using disparate mathematical methods is *simpler* than trying to learn from nature is preposterous. Don't believe me? Let's look at an example. Let's say you have a chess-playing program that operates by classical A.I. methods: the application of various clever data structures and algorithms. Sure, it plays chess well enough to beat a world champion. Congratulations! But what if I wanted to play checkers against it instead? The rules of checkers are quite different from those of chess, so I imagine that to add some world-class checkers-playing ability while retaining your program's chess-playing skill, you'll have to add some complexity to your code base. What if I wanted it to pass the Turing Test too? What if I wanted it to catch a ball? Or write a blog post? Suddenly, we are awash in compounding complexity.

> But what if I wanted to play checkers against it instead?

I suppose the my final cringe came when I saw a quote from Dr. Ray Kurzweil appear in the professor's presentation. I am rather sure that Dr. Kurzweil would be ashamed to appear amongst such rubbish. I will not go into detail here, but Dr. Kurzweil's methods of creating intelligent computer programs are far from the "forget brains and just throw algorithms at the problem" approach. In case you are interested in learning more about his work, I have included a link to his book "How to Create a Mind" below.

Perhaps what really disturbed me was that this class appears to be exactly what I was afraid it would be. In the beginning of his masterful work "On Intelligence," Jeff Hawkins lays out the history of the A.I. movement and critiques its fallacies and shortcomings. What Hawkins witnessed was the A.I. movement in the 1980's, and, evidently, it hasn't changed at all since.

## Resources

**If you are interested in researching a different, more brain-centric view of creating intelligent computer programs, or simply think that it would be nice to have a definition of what intelligence is before attempting to create it, I encourage you to read the books below.**

- [On Intelligence](http://www.amazon.com/Intelligence-Jeff-Hawkins/dp/0805078533/) by [Jeff Hawkins](https://en.wikipedia.org/wiki/Jeff_Hawkins).
- [Numenta White Paper](http://numenta.org/resources/HTM_CorticalLearningAlgorithms.pdf) supplimenting the above text (direct link).
- [How to Create a Mind](http://www.amazon.com/How-Create-Mind-Thought-Revealed/dp/1491518839) by [Dr. Ray Kurzweil](http://www.kurzweilai.net/ray-kurzweil-biography).
