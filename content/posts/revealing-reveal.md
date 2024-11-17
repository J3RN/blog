---
layout: post
title: "Revealing Reveal"
date: 2024-11-17
description: An alternate approach for conveying syntactic information to programmers.
tags:
- software development
- tree-sitter
---

**Disclaimer: A limitation of the work that I'm describing in this post is that it may only be useful to sighted persons and is enhanced by being not colorblind.  This post discusses a tool that displays code within colored boxes.  The current implementation provides titles on the boxes which might be of use to those without (color) vision, but I have not verified it with a screen reader.  I will strive in my future work to create tools that are of use to the (color)blind community.**

Over the last several years, I've been intrigued by *projectional editors*, or editors that are aware of syntactic (and optionally semantic) constraints of what they're being used to create.  There are some reasonably good examples; for instance, rich text fields in websites ultimately generate HTML markup, but their user interfaces hide the markup and instead provide buttons like "**Bold**" that seamlessly insert markup that the user never sees.  However, editors for writing software are notably lacking in this regard.  Sure, modern editors have a litany of helpful features, but few actually prevent you from making trivial syntactic errors such as forgetting a semicolon.

After years of research into this field, I set out to build a projectional editor for code, analogous to [Fructure] (pictured below) (see also: [Andrew Blinn's talk on Fructure at RacketCon]).  "How hard could it be?" I thought.  It turns out: it's pretty hard.

![Racket code partially represented by text in colored boxes.](/images/fructure.png)

Creating a projectional editor was so hard in fact that my first attempt went so sideways as to not be a projectional editor at all, but something totally different: [Reveal].

![Python code represented as text within nested, colored boxes.  Each syntactic construct is the same color (e.g. integers are cyan).](/images/Reveal.png)

A feature of Fructure that I absolutely loved was the way that different syntactic structures were encased in colored boxes.  In the screenshot above, `true` is always in a green box.  The condition is always blue.  This immediately reveals (to sighted, non-colorblind persons) a great deal of information at a glance.  It also triggered an epiphany: text highlighting isn't good enough because it fails to show the *extent* of syntactic forms.

![A JavaScript arrow function in an editor with traditional syntax highlighting.](/images/arrow_function.png)

JavaScript's "arrow functions" became the poster child of this phenomenon for me.  In my work mentoring new developers, this syntax frequently confused them; there is no clear indication that the arguments (`(...)`) and body (`{ ... }`) are necessarily part of the same syntactic structure.

![Reveal, where the JavaScript arrow function is wrapped in a purple box labeled "arrow function".](/images/reveal_arrow_function.png)

The key insight of Reveal is to, like Fructure, *wrap* syntactic structures to show their extent in addition to their "type" (e.g. variable, function, etc), whereas (good) syntax highlighting can only convey the general "type" of text it represents.

## Why Reveal is No Good

Problem solved, right?  Well, not quite.

There are at least two large problems with Reveal; feel free to [tell me more](https://fosstodon.org/@j3rn) as you find/think of them.

1. Multi-line structures look terrible
    ![Reveal showing an Elixir module declaration, but the name of the module is rendered to the left of a large block containing the body of the module.](/images/reveal_multi_line.png)
	
	This issue is, conceivably, solvable.  My intention here is to make multi-line syntactic structures wrap such that the rendered Reveal version of the code closely resembles the text version.
	
	![A mockup where an Elixir function's body is wrapped such that its name is above the body.](/images/reveal_concept2.png)
	(Pardon the really garish colors, this was a "quick and dirty" mockup)
	
	I came to believe that HTML and CSS, the tools currently rendering Reveal, are not capable of rending this, and so I began work on [an SVG renderer](https://github.com/J3RN/reveal/blob/main/svg_renderer.js).  Work on that has been limited.
	
2. Information overload

    In conversations with people about Reveal, I usually frame its utility as "conveying a greater amount of information" than, say, conventional syntax highlighting.  As mentioned before, Reveal conveys strictly more information by conveying the extent—in addition to the "type"—of syntactic structures.  This is, evidently, a double-edged sword.  Consider again the JavaScript example from earlier:
	
	![Reveal, where the JavaScript arrow function is wrapped in a purple box labeled "arrow function".  The body of the arrow function is wrapped in alternating yellow and green boxes.](/images/reveal_arrow_function.png)
	
	The alternating yellow and green boxes in the function body create a great deal of visual noise.  Here, the yellow boxes represent function application (grouping a callee with arguments) and the green boxes represent field access (e.g. `document.querySelector`).  I am reminded of the saying "If everything is important, nothing is important".  Here, the JavaScript is syntactically dense, resulting in a large volume of syntax annotation.  But is all the annotation worthwhile?  Are these yellow and green boxes helping anyone, or just getting in the way?  Can we simply train our brains to ignore the information we don't care about but not the information we need?
	
	These are worthy questions, and I don't have answers to any of them.
	
In any event, I hope my work and discussion thereof helps you in your endeavors.  And, if you want to finish my SVG renderer, [pull requests are welcome](https://github.com/J3RN/reveal)! 🙏

[Reveal]: https://j3rn.com/reveal
[Andrew Blinn's talk on Fructure at RacketCon]: https://www.youtube.com/watch?v=CnbVCNIh1NA
[Fructure]: https://github.com/disconcision/fructure
