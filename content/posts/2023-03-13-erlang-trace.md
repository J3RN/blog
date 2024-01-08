---
title: "Utilizing Erlang's Tracing System"
layout: post
date: 2023-03-13
description: TBD
draft: true
tags:
- erlang
- tracing
---

`erlang:trace` seems like `dbg:p`.  Do we need to do a step before this? (I don't think so)
It returns an integer?

```elixir
:erlang.trace(:all, true, [:call])
```
```
:erlang.trace_pattern({String, :_, :_}, [{:_, [], [{:return_trace}]}], [:local])
```

Ah, specify which functions with `erlang:trace_pattern/3`
