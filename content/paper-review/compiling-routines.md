---
layout: post
title: "Paper Review: *Compiling Routines* by Richard K. Ridgway, 1952"
date: 2025-06-17
description: |
    *Compiling Routines* is a short paper that describes one of the first compilers, termed "A-0".
tags:
  - paper review
---

## [Compiling Routines](https://dl.acm.org/doi/pdf/10.1145/800259.808980)
### Richard K. Ridgway, 1952

*Compiling Routines* is a short paper (3 paged of text, 2 pages of figures) that describes one of the first compilers, termed "A-0".  A-0 is also referred to as "the antique compiler" in the paper as, apparently, newer compilers had been developed.

Speaking of compilers, this paper appears to invent the term "compiler", or at least introduces it in a way that implies the reader would not already be familiar.  Specifically, according to Ridgway:

> A compiler looks up subroutines, adjusts them, and assembles them, as a complete program.

This feels a bit different from how one might describe a compiler today, e.g. as a program that translates one language (e.g. Rust) into another language (e.g. machine code).  However, the name "compiler" does seem much better suited to a program that assembles a compilation of subroutines.

The paper also provides this insight into the day-to-day work of programmers in the early 1950s:

> By the conventional method:
> 1) The programmer analyzed the problem in twenty minutes.
> 2) Four hundred and eight minutes were required to prepare and write the program.
> 3) Checking the program required 240 man-minutes.
> 4) Forty-five minutes were required to transcribe the program on tape.
> 5) Twenty minutes were devoted to typing out the tape.
> 6) Proofreading the tape required forty minutes.
> 7) Sixteen minutes were needed to correct the tape on UNIVAC.
> 8) Fifteen minutes were spent checking the program on UNIVAC.
> 9) Four minutes were required to solve the problem on UNIVAC.
>
> Thus, 740 programmer minutes, 35 UNIVAC minutes, and 105 ancillary-manpower-and-equipment minutes were required to program and solve the problem.

![The UNISERVO, the I/O tape drive for the UNIVAC](/images/uniservo.jpg)

Imagine figuring out a solution to a programming problem and needing an additional *12 hours* to write and run the program!

The UNIVAC, the machine in this paper, was notably also the machine Remington Rand built for the U.S. Census Bureau, delivered the year this was published.

![The UNIVAC at the U.S. Census Bureau](/images/Univac_I.jpg)

A fuller connection with modern compilers comes on the second page of the paper:

> The first diagram represents the method of problem solution conventionally employed.  In phase one, the programmer analyzes the problem, breaking it down into arithmetic steps which are reduced to the instruction code of the computer.

> If a compiler is used, there are three phases to the problem solution, Fig. 2.  In phase one, the programmer analyzes the problem, and breaks it down into steps, each of which can be performed by a subroutine.

> The basic element of the compiled program is the subroutine.

The change from programming by writing machine code by hand to programming by arranging named subroutines is, in effect, the creation of a programming language, though Ridgway does not use this terminology or appear to think about the work in that way.  Things get a bit more interesting on the third and final page:

> In mathematics, a "function" can be defined as "a law which transforms one set into another set.  If the first set be given, the second set is determined".

> Hence, it follows that a compiler can deal with operations that fall under the definition previously given.  Thus, the compiler manipulates functions in symbolic form rather than numerical data.  The implications of this line of thought are not completely apprehended at present.

It is nice to see that Ridgway is thinking of the correspondence between subroutines and mathematical functions at this early era of computer history.  Indeed, this idea of a compiler as a sort of meta-computation is an interesting one, and does not directly correspond with modern conceptions of compilers that I have seen (namely, as automatic translators), though the idea of computing over functions is very commonplace today (e.g. first-class functions).

In all, *Compiling Routines* is a succinct look into the state of programming in 1952, and how the field was set upon the trajectory that got it to where it is today.

