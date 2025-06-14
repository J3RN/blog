---
layout: post
title: "Katahdin Part 1: The Idea"
date: 2024-12-08
description: 
tags:
- language design
- functional programming
- linux
draft: true
---

Modern computers are marketed to us as appliances.  When you buy a household appliance, like a washing machine or a microwave, the intention of the manufacturer is that you push the buttons and turn the knobs provided to you, all in accordance with machine's manual.  How it works is none of your business; a trade secret even.  Opening the case and modifying the internals is very strongly discouraged (for household appliances, often for good reasons).

Computers, the physical machines, are a bit of a mix.  On the one hand, there are machines such as Macbooks[^1], which are not intended for you to ever open and inspect or repair.  Apple isn't the only company with this mentality, you can find devices with the same constraints from Dell, Lenovo, and almost any major other manufacturer.  On the other hand, there are the DIY, build-it-yourself desktop computers, where you buy each component (CPU, RAM, power supply, etc) individually, and spend a couple hours meticulously piecing it together.  For laptops, we happily now have the [MNT Reform] and [Framework] that encourage you to build your machine and look under the hood.

![A Framework laptop open, with the motherboard being placed inside](/images/framework.png)
<small>Photo courtesy of Framework Computer Inc</small>

Fundamentally, I want users to have control over their computers in order to use them to accomplish their goals.  Users today are constrained by today's software offerings, be it web browsers, games, desktop environments, or operating systems.  Often times these are proprietary, closed-source projects that do not allow any user customization.  Even well-intentioned open source products often constrain users by requiring them to set up a development environment in order to customize or extend the software.  Perhaps only a personal opinion, but I find C++, the language that most Linux GUI software is written in today, to be difficult to work with.

My dream is for users to be able to write their own software, easily.  No complicated development environment setup, no having to find the right system-level dependencies, etc.  I also want the user to be able to easily extend and customize  software that they do not write themselves—the operating system itself, the default user interface that it comes with, and any included applications (e.g. a web browser).

<!-- Emacs -->
Much of this vision is inspired by Emacs.  Emacs is, at core, a Lisp interpreter.  The various parts of the editor that the user interacted with—files, buffers, windows—are implemented (at least partially) in Lisp, and their definitions can be changed at any time by the user executing some Lisp (Lisp can be entered into a prompt, loaded from a buffer, or loaded from a file).

<!-- Editor, Desktop, Operating System -->
My approach is to start small, and go from there.  First, I want to create an application that is like Emacs.  At core there's an interpreter (or compiler+hot loading VM, obscured) that has a few built-in functions for doing things one would need in an IDE, such as highlighting text, splitting windows, triggering compilation, etc.

Once the concept is proven, I would move on to the next layer down: the desktop environment, as we originally discussed.  The base "interpreter" would have different built-in functions, specifically for making new windows, positioning windows, and creating and laying out the "chrome" of the desktop itself—tool bars, docks, things like that.

Lastly would be to create an operating system.  Here, the interpreter would need built-ins for things like managing hardware devices.

I'm thinking of making a Linux desktop which essentially functions as an code interpreter.  Any built-in functionality (e.g. UI for connecting to Wi-Fi, search bar, etc) would be implemented in the native language for the interpreter, and the source for those items would come with the desktop (more or less out of necessity, since—as I mentioned—the desktop is an interpreter at core).  This would allow the user to easily customize their desktop without having to find the source, install build tooling, etc.

<!-- Smalltalk, Self, with types? -->

[^1]: An apt example, as the Apple II was one of the first home computers sold as an appliance and not a kit.

[MNT Reform]: https://mntre.com
[Framework]: https://frame.work
