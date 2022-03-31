---
layout: post
title: "Readability vs Comprehensibility"
date: 2022-03-31
description: |
  For me, clarity is composed of two smaller ideas: _readability_ and _comprehensibility_. Neither of these words have widely-accepted definitions in software engineering, but in this post I'll elaborate what I mean by each.
draft: true
tags:
- language design
- software development
- elixir
- ruby
- haskell
---

For me, clarity is composed of two smaller ideas: _readability_ and _comprehensibility_. Neither of these words have widely-accepted definitions in software engineering. However, for the purposes of this article:

<dl>
<dt>readability</dt>
<dd>ability for a reader to recognize the syntactic constructs in a body of code.</dd>
<dt>comprehensibility</dt>
<dd>ability for a reader to discern the what a body of code does at a high level.</dd>
</dl>

#### Readability

The LISP family of programming languages are the pinnacle of _readability_.

Consider that in Clojure, a LISP, addition looks like this:
```
(+ 1 2)
```
Printing to the console looks like this:
```
(println "Hello, World!")
```
"If" expressions look like this:
```
(if true :was-true :was-false)
```

In each case, the expression has the form `(<function> <arg1> <arg2> ...)`. That's it! Now you can read any line of LISP.

Contrast this with a language such as C where addition looks like this:
```
1 + 2
```
Printing to the console looks like this:
```
printf("Hello, World!");
```
"If" statements look like this:
```
if (condition) {
  // perform "then" actions
} else {
  // perform "else" actions
}
```

Each of these statements and expressions uses a different syntax (infix, prefix, multi-arm conditional), and these are not the only forms in the language either. Many beginner C programmers believe they have learned all the syntax of the language only to be bewildered by ternary expressions.

#### Comprehensibility

Readability, however, is often in contention with comprehensibility. Let's compare two recursive implementations of a factorial function:

**Clojure**
```clojure
(defn fac [x]
  (if (<= x 1)
    1
    (* x (fac (- x 1)))))
```

**Haskell**[^3]
```haskell
fac x
  | x < 1 = 1
  | otherwise = x * fac (x - 1)
```

The uniformity of the syntax of expressions in LISPs such as Clojure can impair the reader's ability to quickly determine what purpose a given expression is serving. Furthermore, the prefix notation of LISPs (e.g. `< a b`) can make it difficult for the reader to decipher expressions where they are used to seeing infix notation (e.g. `a < b`).

The readability of Haskell's syntax is, for me, usually quite low due to its wide variety. Once you've learned most of the syntactic forms for pure functions, you discover that there's a whole _other_ syntax for effect monads (the `do` syntax). In the above example, there are several different syntactic forms present:
- function definition
- guards
- infix function invocation
- prefix function invocation

If you were wondering, "otherwise" is a keyword, like "else" in many languages. I find "otherwise" more intuitive as I more often say "otherwise" than "else".

The ability of Haskell's syntax to clearly convey mathematical concepts is in part due to [ISWIM]'s influence on the language (by way of ML), where similarity to mathematical notation was one of ISWIM's design goals. Sadly, Haskell's syntax comprehensibility for domains other than mathematics is not so great due to it's inside-out and right-to-left evaluation style.

While the syntax of a language can make a significant impact on the comprehensibility of applications written in that language, a great deal of the burden of writing comprehensible code falls on the author in the form of application architecture and code organization. For instance, consider this [500+ line Go function in the Lightning Network Daemon]. How long would it take you to understand or explain to someone else what this function does? I'm guessing quite a while. By splitting large procedures into small, independently understandable functions, an author can more clearly convey what a piece of code is doing.

[^3]: Personally I find a non-recursive Haskell implementation that uses ranges to be the most comprehensible: `let fac x = product [1..x]`
