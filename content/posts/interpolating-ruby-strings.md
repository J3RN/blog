---
layout: post
title: "Interpolating Ruby Strings"
date: 2014-12-19 22:14:25 -0500
description: |
  Sometimes I want string templates that I can pass around. In this post, I discuss how this can be accomplished in Ruby.
comments: true
tags:
- ruby
- interpolation
aliases:
- /posts/2014-12-19-interpolating-ruby-strings
---

Let's talk about interpolation. Sure, you probably know the Ruby string interpolation syntax:

```ruby
arg = "world"
puts "Hello #{arg}!" #=> "Hello world!"
```

This syntax is pretty useful. However, it has it's limitations. Let's say that we are generating strings that are almost the same each time, but with each, one word changes. How are you going to do this now?

My first inclination was something like this:

```ruby
template = "Today the weather is \#{ weather }"
weather_strings = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_strings[weather] = template
end
```

It's not hard to see why `weather` wasn't interpolated into the strings. Well, it occurred to me shortly thereafter that ERB might be the right way to go with this.

First try with ERB:

```ruby
template = "Today the weather is <%= weather %>"
weather_strings = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_strings[weather] = ERB.new(template).result
end
```

This, however, resulted in the following error:

```bash
NameError - undefined local variable or method `weather' for main:Object:
```

After some searching about on the internet, it appears that there is an ever-present `binding` variable that is an instance of the `Binding` class. Strange as that was, it seems to be needed by `ERB#result` to properly interpolate variables.  I also discovered that we can also get by with only building the ERB instance once:

```ruby
template = ERB.new("Today the weather is <%= weather %>")
weather_strings = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_strings[weather] = template.result(binding)
end
```

Another approach, if you don't want to pass ERB *all* variables in your `binding` is to use `result_with_hash` (using a Hash instead of a Binding):

```ruby
template = ERB.new("Today the weather is <%= weather %>")
weather_strings = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_strings[weather] = template.result_with_hash({weather: weather})
end
```

Not very intuitive, but it works!
