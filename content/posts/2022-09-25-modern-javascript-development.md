---
title: "Modern JavaScript Development: Indistinguishable from Parody"
layout: post
date: 2022-09-25
description: A story about a time I tried to accomplish something conceptually rather simple and succeeded only in wasting several hours of my life.
draft: true
tags:
- javascript
- node.js
- webpack
- esbuild
---

[The prolific crypto skeptic Molly White](https://twitter.com/molly0xFFF) will occasionally use the phrase "indistinguishable from parody" when annotating posts by crypto true believers, and I've taken a liking to it and so am using it here.  In a way it's a rephrasing of [Poe's Law](https://en.wikipedia.org/wiki/Poe%27s_law)—whereas Poe meant that peoples' parody will be mistaken for honest belief (namely in Creationism), I believe that White is expressing that peoples' expressions of honest belief could be mistaken for parody—each equally indicative of our world where corners of society have become fully detached from the everyday reality of the rest of us.

Let's talk about JavaScript.

JavaScript has always been something of an ugly duckling programming language.  Once envisioned by its creator (Brendan Eich[^1]) as [an in-browser dialect of Scheme](https://brendaneich.com/2008/04/popularity/), it was born an unfortunate mish-mash.

> I’m not proud, but I’m happy that I chose Scheme-ish first-class functions and Self-ish (albeit singular) prototypes as the main ingredients. The Java influences, especially y2k Date bugs but also the primitive vs. object distinction (e.g., string vs. String), were unfortunate.

- Brendan Eich

<aside>

#### Self-ish prototypes

There have been few languages with built-in prototypical inheritance; Self (as mentioned), [Io](https://iolanguage.org/), arguably [Lua](https://www.lua.org/), and a few others.  Of these, none have achieved mainstream acceptance (Lua is probably the most popular after JavaScript itself), and [ES6 introduced the `class` keyword and method syntax in an apparent  attempt to falsify it's true inheritance system](https://www.toptal.com/javascript/es6-class-chaos-keeps-js-developer-up#the-trouble-with-es6-classes).  If popularity and adoption are indicative of goodness, perhaps the _only_ good part of the design was first-class functions.
</aside>

**TODO** `create-react-app` generates app with 6 high severity vulnerabilities

**TODO** Running `npm audit fix --force` upgrades you to 34 high severity vulnerabilities and 3 "critical"

**TODO** CRA uses
  - left-pad
  - flatten ("use a utility library")

**TODO** tkat0's tutorial just doesn't work

**TODO** Webpack 5 has built-in WASM loading but CRA uses Webpack 4

**TODO** Ballercat's `wasm-loader` references a `loaders` key that doesn't exist; doesn't do anything anyways (perhaps the nested `loaders` key is also erroneous?)

[^1]: [The guy who resigned as Mozilla's CEO after a whopping eleven days over backlash from his support of an attempt to ban gay marriage in California](https://www.theverge.com/2014/4/3/5579516/outfoxed-how-protests-forced-mozillas-ceo-to-resign-in-11-days)
