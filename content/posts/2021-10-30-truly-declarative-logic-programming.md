---
layout: post
title: "Truly Declarative: Logic Programming"
date: 2021-10-30T10:21:19-04:00
description: |
  I solve a simple math problem with three languages representing three paradigms which I see placed along the sliding scale of "declarativeness".
tags:
- logic programming
- prolog
- haskell
---

While reading through Miran Lipovača's excellent [_Learn You a Haskell_](http://learnyouahaskell.com/), I came across the following problem:

> Which right triangle that has integers for all sides and all sides equal to or smaller than 10 has a perimeter of 24?

To demonstrate some properties of different programming languages, I'll solve this problem in three languages embodying three different paradigms:
- C (imperative and procedural)
- Haskell (functional)
- Prolog (logic and constraint)

## C

The general approach I'll be taking here is to generate every set of possible lengths fitting the constraints (namely being integers between 1 and 10) and then check to see if the other two conditions are met (being a right triangle, having a perimeter of 24).

Let's start with the side-length piece[^1]:
```C
void right_triangle() {
  int a, b, c;

  for (c = 1; c <= 10; c++) {
    for (b = 1; b <= 10; b++) {
      for (a = 1; a <= 10; a++) {
        # TODO: Perform perimeter and right triangle checks
        printf("[%i, %i, %i]\n", a, b, c);
      }
    }
  }
}
```



Running this code shows not only every possible set of sides, but every possible _permutation_ of every possible set of side lengths! That's a bit much. Let's decide that `c` is the hypotenuse, `b` is the next longest side, and `a` is that last side that is at most equal in length to `b`. These constraints also reduce our search space, making the code run faster!

```C
void right_triangle() {
  int a, b, c;

  for (c = 1; c <= 10; c++) {
    for (b = 1; b < c; b++) {
      for (a = 1; a <= b; a++) {
        # TODO: Perform perimeter and right triangle checks
        printf("[%i, %i, %i]\n", a, b, c);
      }
    }
  }
}
```

OK, great. Now let's check our perimeter and right triangle constraints. The former is simply a matter of adding up the side lengths, nothing special there. For checking that our side lengths create a right triangle, Pythagorus tells us that, given that the hypotenuse of a triangle is represented by "c" and the other two sides by "a" and "b", the side lengths must follow the rule a² + b² = c².

Let's add those checks:

```C
void right_triangle() {
  int a, b, c;

  for (c = 1; c <= 10; c++) {
    for (b = 1; b <= c; b++) {
      for (a = 1; a <= b; a++) {
        if (a + b + c == 24 && pow(a, 2) + pow(b, 2) == pow(c, 2)) {
          printf("[%i, %i, %i]\n", a, b, c);
        }
      }
    }
  }
}
```

That's it! When we call this function, the solution to the problem (`[6, 8, 10]`) is printed to the console.

Try it for yourself!

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/C-solution?lite=true"></iframe>

## Haskell

In Haskell, the most analogous way to solve this problem is via a "list comprehension". This construct exists in other languages, and often looks quite similar[^2].

Let's break down how list comprehensions work. They have three parts:
1. Generator(s)
2. Filter(s)
3. Mapper

### Generators

Let's start with the generators. Using our optimization from the C solution, our first generator is `c <- [1..10]`, which will give the binding `c` each value between `1` and `10`, followed by `b <- [1..c]`, which will give `b` each value between `1` and `c`, and ending with `a <- [1..b]`, which will give `a` each value between `1` and `b`.

The comprehension with only our generators (and technically also a mapper, but we'll get to that in a moment) looks like this:
```haskell
> [(a, b, c) | c <- [1..10], b <- [1..c], a <- [1..b]]
[(1,1,1),(1,1,2),(1,2,2),(2,2,2),(1,1,3),(1,2,3),(2,2,3),(1,3,3) ...
```
(the result is very long, and has been truncated)

See for yourself!

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/Haskell-Generators?lite=true"></iframe>

### Filters

Now we need to filter out all sets of sides that don't form a right triangle or don't sum to 24. Filters go after generators (though I think the two can be interspersed), and are simply boolean expressions. For instance, the perimeter filter is the boolean expression `a + b + c == 24` and the right triangle filter is the boolean expression `a^2 + b^2 == c^2`.[^3]

This gives us:
```haskell
> [(a, b, c) | c <- [1..10], b <- [1..c], a <- [1..b], a + b + c == 24, a^2 + b^2 == c^2]
[(6,8,10)]
```

There's our solution! Try it!

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/Haskell-Triangle-Solution?lite=true"></iframe>

### Mappers

But wait, there's more! Did you notice the `(a, b, c)` bit at the beginning of each comprehension? What happens if we changed that to something different?

```haskell
> [(a * b) / 2 | c <- [1..10], b <- [1..c], a <- [1..b], a + b + c == 24, a^2 + b^2 == c^2]
[24.0]
```

I changed the expression `(a, b, c)` to the expression `(a * b) / 2`, which gives us the area of the triangle instead of a tuple of the lengths of its sides. But why call it a mapper? Well, that's because each set of generated values that make it past the filters are given to the mapper to be turned into some kind of final result. In the case of this problem, there's only one set of values that make it past the filters (`c` being 10, `b` being 8, and `a` being 6) and so only that one set of values gets "mapped".

See for yourself!

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/Haskell-Mapper?lite=true"></iframe>

## Prolog

The Prolog solution that we'll write will be profoundly different from the previous two. While the Haskell syntax is far more concise and expressive than the C syntax, the solutions in both languages follow essentially the same two steps:
1. Generate all possible combinations of side lengths
2. Check each set of side lengths to see if it's the solution.

However, Prolog doesn't have the facilities to do these things, at least not in the same way. A Prolog program consists of two elements: facts and rules.

### Facts

Facts are exactly what they sound like. For instance, the following fact states that `mike` is the father of `john`:

```Prolog
father(mike, john).
```

The term `father` in the fact above is referred to as a "relation" because it describes the relationship between `mike` and `john`.

Having defined a fact, we can ask Prolog about it[^4]:
```Prolog
?- father(mike, john).
true.
?- father(A, john).
A = mike.
```

(_`?-` is the Prolog REPL prompt, where you can make queries_)

Prolog can tell us if a statement is true, or how we might make it true. Nifty!

Here, try it for yourself!

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/Prolog-Fact-Example?lite=true"></iframe>

### Rules

Rules represent ways in which we can extend our existing facts or rules to derive more information. For instance, below is an `ancestor` rule that builds upon our `father` relation. It can tell us if `A` is a paternal ancestor of `B`:

```Prolog
ancestor(A, B) :-
  father(A, B);
  father(C, B), ancestor(A, C).
```

In Prolog, `;` is read as "or" and `,` is read as "and", making the above rule:
> A is an ancestor of B if A is the father of B or if C is the father of B and A is an ancestor of C.


Try it! When Prolog returns a result but doesn't create a new prompt, this indicates that there may be more than one result, and Prolog wants you to tell it whether you want it to try to find the next answer or stop. To tell Prolog to try to find another solution, type `;`. To tell Prolog to stop searching for solutions, type `.`.

<iframe frameborder="0" width="100%" height="500px" src="https://replit.com/@J3RN/Prolog-Rule-Example?lite=true"></iframe>

### Back to Triangles

**Interesting! But what does this all this have to do with triangles?** Well, nothing, other than that relations can also be used to express _constraints_ (since 1982's Prolog II). For instance, the following line states that `A` must be between 1 and 10 (inclusive):

```Prolog
?- between(1, 10, A).
A = 1 ;
A = 2 ;
A = 3 ;
A = 4 ;
A = 5 ;
A = 6 ;
A = 7 ;
A = 8 ;
A = 9 ;
A = 10.
```

Note that I didn't have to define `between/3`, it is built-in to Prolog.

In fact, we can phrase our entire problem as a set of constraints on the triangle's sides!
1. The lengths of the three sides must be between 1 and 10.
2. The perimeter of the triangle (the sum of the sides) must equal 24.
3. The sides must form a right triangle.

Therefore, our complete Prolog solution is this:
```Prolog
solution(A, B, C) :-
    between(1, 10, C), between(1, C, B), between(1, B, A),
    24 is A + B + C,
    0 is A^2 + B^2 - C^2.
```
(_Note that I also included our additional constraints on A and B to preclude permutations here._)

Queried like so:
```Prolog
?- solution(A, B, C).
A = 6,
B = 8,
C = 10
```

Try it!

<iframe frameborder="0" height="500px" width="100%" src="https://replit.com/@J3RN/Prolog-Triangle-Solution?lite=true"></iframe>

Now, you've probably noticed something strange about some of these rules: Namely, that I used `is` instead of `=` whenever I had to do math. This is, sadly, just part of the way Prolog is; math is largely an afterthought. Doubly so, because the left side of `is` is required to be a number or variable, but not an expression. For instance, the following is invalid:
```Prolog
C**2 is A**2 + B**2.
```

Pretty gnarly, isn't it?

## Conclusions

Of the above three, I'm particularly fond of the Prolog solution despite it's gnarly math limitations. **_"Why?"_** you ask? Well, because the Prolog solution is the most _declarative_. [Ward Cunningham defines declarative programming as the following](https://wiki.c2.com/?DeclarativeProgramming):

> Programming where problems are described, or conditions on a solution are described, and the computer finds a solution.

In the Prolog solution, as opposed to the C and Haskell solutions, I was not required to give any information about _how_ to go about solving this problem; Prolog was able to do that on it's own. Heck, it may have created 220 threads to check each possible combination of side lengths simultaneously or it may have just tried numbers at random; it's really no concern of mine. And I _love_ that.

However, Prolog, as we've seen, isn't the most practical or pretty language. While it was well-suited to the problem that I addressed here, it is not well-suited to every problem, perhaps not even a _majority_ of problems. Additionally, Prolog intends to be a standalone programming language and most distributions (such as [SWI-Prolog](https://www.swi-prolog.org/) or [GNU Prolog](http://www.gprolog.org/)) include rules for performing side-effects (such as writing to the console), which don't really make sense in logic or constraint programming.

With that said, there is a promising new trend of logic and constraint DSLs that can be used within functional or imperative languages, such as [miniKanren](http://minikanren.org/) and [Erlog](https://github.com/rvirding/erlog). Introducing these paradigms inside of existing languages (which commonly already feature DSLs for purposes such as building regular expressions or constructing SQL queries) allows developers to reach for them when they provide the most elegant solution and ignore them otherwise. This, I feel, is the future for logic and constraint languages, and I personally can't wait!

[^1]: While I declared my variables at the top of my function, I think modern C allows for you to write variable declarations inline in `for` loops (e.g. `for (int c = 1; ...`). I'm old-fashioned though.
[^2]:
    Equivalent solution in Erlang:
    ```erlang
    right_triangles() ->
      [{A, B, C} || C <- lists:seq(1, 10), B <- lists:seq(1, C), A <- lists:seq(1, B), math:pow(A, 2) + math:pow(B, 2) == math:pow(C, 2), A + B + C == 24].
    ```
  
    The Erlang syntax is much more verbose, but fundamentally we've only traded `(a, b, c)` for `{A, B, C}`, `|` for `||`, `[a..b]` for `lists:seq(A, B)`, and `a^b` for `math:pow(A, B)`.

[^3]: `^` is the exponentiation operator in Haskell
[^4]: In Prolog, terms starting with a lowercase letter are atoms and terms starting with an uppercase letter are variables.
