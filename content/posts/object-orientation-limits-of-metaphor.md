---
layout: post
title: Object Orientation and the Limits of Metaphor
date: 2025-04-13
description: Reflections on the paradigm and the history of programming languages
tags:
  - erlang
  - ruby
  - smalltalk
  - object orientation
  - language design
  - software development
---

Roughly ten years ago, I was very excited about object orientation.  Which made me somewhat late to the party; glowing pronouncements of how object orientation was going to solve the software complexity crisis had flooded industry publications some two decades prior.  Regardless, in 2014, I was trying to explain to a friend why objects were so grand.

"Objects model the real world," I said, grabbing a pencil off the table and holding it up.  "For instance, I could create a Pencil class, representing the Platonic idea of Pencil, and this would be an instance of it."  I, like any modern person, did not actually subscribe to Platonism.  "And it's able to *do* stuff, which we refer to as 'methods'.  For instance," I said, scribbling on a piece of paper, "it might have a 'write' method, for writing things."

And in that moment, I realized that there was something deeply wrong with my example, because in the real world, the *pencil* wasn't writing, *I* was writing with the pencil.

An "object", in the programming language sense, refers to a construct where data (often referred to as "state") is bundled together with functionality (typically referred to as "methods").  When you "pass" an object from one section of code to another, the data and functionality go together.  The programming language Simula, first developed in 1962, is widely considered the originator of "object orientation"[^1].  To a greater of lesser extent, object orientation seems to be a great fit for simulation!  In a simulation, you model a set of *agents* (hawks, doves, wolves, deer, people, etc) that
- have some state of being
- are able to do different things
They are, in this way, different from a pencil which has a state of being, but isn't necessarily able to *do* anything on its own.  It does not act, it is acted upon.  It is not an *agent*.

Simula went on influence Smalltalk, which was first developed by Alan Kay at Xerox Parc, which implemented the idea of "message passing".  That is, when one object (let's say, an object representing me) wants to communicate with another object (a pencil), it "sends it a message", which the recipient can choose how to process.

Message passing makes metaphors somewhat more complicated.  When a wolf eats a deer in the real, physical world, it does not "send it a message" that it is eating it, the wolf acts upon the deer directly.  However in the polite world of our software systems (or so we like to imagine), wolves are not predating on deer, rather our assorted components are working together cooperatively, sending messages through the post or office memorandums or, perhaps, by [calling one another on the telephone](https://www.youtube.com/watch?v=xrIjfIjssLE).

Another idea that Smalltalk pioneered is that *everything* ought to be an object, even primitives such as characters or integers.  In Smalltalk, to add two numbers, you would need to send a message to one number to add itself to the other number.  This idea produces a system that is simple and orderly in its philosophy, but lacks little, if any, correspondence to physical objects in the real world.

You might argue that conceptual computer processes need not resemble the real world, but I would argue that our human minds, evolved to find nuts in a forest, strive when they are able to map abstract concepts to physical phenomena.

In the mid-1980s, a small team at Ericsson was designing a new programming language for use in telephone exchanges.  Their language, Erlang, was truly set apart by its interpretation of "message-passing concurrency".  In Erlang's model, individual "processes" have their own state, work independently, and, if they need to communicate, one process could send a message to another process's "mailbox".  In this way, an Erlang *process* is very much like a Smalltalk object.  In fact, Erlang co-creator Joe Armstrong once said:

> Erlang has got all these things. It's got isolation, it's got polymorphism and it's got pure messaging. From that point of view, we might say it's the only object oriented language 

It's important, at least for my purposes here, to emphasize that if we say that Erlang processes *are* objects, a typical Erlang program has *far fewer* objects than an equivalent program in a conventional object-oriented language like Smalltalk, Ruby, or Java.  In Erlang, `1` is not a process, it's just a piece of data.  You cannot send `1` a message.

Revisiting the motivating example, if we were to model that conversation in Erlang, I would be a process and my friend would be a process, because we are independent *agents* in the world, with our own states of being and abilities to perform actions.  I would be sending my friend messages (my spoken statements) that they could choose to deal with however they want.  When I write with a pencil, it is *me*, an *agent*, a process, doing it, and the pencil is just inert data to be acted upon.

Fans of functional programming will frequently rail against object orientation and its perceived contradictions and complexities, and I have been guilty of this from time to time.  But perhaps the problem hasn't been objects, themselves, but taking the metaphor too far; of making *everything* an object.  Perhaps what software systems need are *just enough* objects.

Further reading:
- [Paul Graham's thoughts on OO](https://paulgraham.com/noop.html)
- [Jonathan Rees' reply to Graham](https://mumble.net/~jar/articles/oo.html)

[^1]: The term "object" referred to a few different things prior to Simula.  The idea of classes and instances existed in Ivan Sutherland's *Sketchpad* project from a few years prior.
