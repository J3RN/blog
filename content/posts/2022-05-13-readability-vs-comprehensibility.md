---
layout: post
title: "Readability vs Comprehensibility"
date: 2022-09-04
description: |
  For me, clarity is composed of two smaller ideas: _readability_ and _comprehensibility_.  Neither of these words have widely-accepted definitions in software engineering, but in this post I'll elaborate what I mean by each.
tags:
- language design
- software development
- clojure
- C
---

For me, clarity is composed of two smaller ideas: _readability_ and _comprehensibility_.  Neither of these words have widely-accepted definitions in software engineering, so for the purposes of this article, I will define them like so:

<dl>
<dt>readability</dt>
<dd>ability for a reader to recognize the syntactic constructs in a body of code.</dd>
<dt>comprehensibility</dt>
<dd>ability for a reader to discern the what a body of code does at a high level.</dd>
</dl>

## Readability

The LISP family of programming languages have reached the pinnacle of _readability_.

Consider that in Clojure (a LISP) addition looks like this:
```clojure
(+ 1 2)
```
Printing to the console looks like this:
```clojure
(println "Hello, World!")
```
"If" expressions look like this:
```clojure
(if true :was-true :was-false)
```

In each case, the expression has the form `(<function> <arg1> <arg2> ...)`.  That's it!  Now you can parse any line of LISP.

Contrast this with a language such as C where addition looks like this:
```C
1 + 2
```
Printing to the console looks like this:
```C
printf("Hello, World!");
```
"If" statements look like this:
```C
if (condition) {
  // perform "then" actions
} else {
  // perform "else" actions
}
```

Each of these statements and expressions uses a different syntax (infix operator, function invocation, multi-arm conditional), and these are not the only forms in the language either.  After years of study, many C programmers believe they have learned all the syntax of the language only to be bewildered by ternary expressions.

## Comprehensibility

Readability, however, is often in contention with comprehensibility.  Let's compare two recursive implementations of a factorial function:

**Clojure**
```clojure
(defn factorial [x]
  (if (<= x 1)
    1
    (* x (factorial (- x 1)))))
```

**C**[^1]
```C
int factorial(int x) {
  if (x <= 1) return 1;
  else return x * factorial(x - 1);
}
```

The uniformity of the syntax of expressions in LISPs such as Clojure can impair the reader's ability to quickly determine the meaning of a given expression.  For instance, the prefix notation of LISPs applied to boolean and arithmetic operators (e.g. `< a b`) can be difficult for the reader to decipher when they are used to seeing infix notation (e.g. `a < b`).

By contrast, the diversity of syntactic forms in the presented C code (discussed above), with the exception of the noisy inline type specifications, lead to a very comprehensible function (at least in my opinion).  With only a little imagination, the function's body can be read as the English sentence "If x is less than one, return one; else return x times the factorial of x minus one."  If I had to explain to someone how this function worked, that is almost precisely the language that I would use to do so (although I would probably say "otherwise" instead of "else").

However, if I had to explain which _syntactic forms_ were utilized in the C function, I would have substantially more difficulty (infix operators, explicit return, braceless conditional?).

## Wrap Up

While the syntax of a language can make a significant impact on the readability and comprehensibility of programs written in that language, a great deal of the burden of writing comprehensible code falls on the author in the form of code style, application architecture, and code organization.  For instance, consider this [~200 line Go function in the Lightning Network Daemon].  How long would it take you to understand or explain how this function works?  I'm guessing quite a while.  As such, I hope to explore further how authors can make their codebases more comprehensible.

[~200 line Go function in the Lightning Network Daemon]: https://github.com/lightningnetwork/lnd/blob/9d04b0c3d9577b2ab8c253e140220e80181619b8/rpcserver.go#L5033-L5244 

[^1]: If you write C, you probably have your own preferred style and this is certainly not it. Bear with me here.
