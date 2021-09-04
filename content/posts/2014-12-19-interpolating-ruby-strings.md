---
layout: post
title: "Interpolating Ruby Strings"
date: 2014-12-19 22:14:25 -0500
comments: true
tags:
- ruby
- interpolation
---

Let's talk about interpolation. Sure, you probably know the Ruby string interpolation syntax:

```ruby
arg = "world"
puts "Hello #{arg}!" #=> "Hello world!"
```

This syntax is pretty useful. However, it has it's limitations. Let's say that we are generating strings that are almost the same each time, but with each, one word changes. How are you going to do this now?

My first inclination was something like this:

```ruby
line = "Today the weather is \#{ weather }"
weather_conditions = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_conditions[weather] = %r/line/i
end
```

It's not hard to see why this didn't work. Well, it occurred to me shortly thereafter that ERB might be the right way to go with this. 

First try with ERB:

```ruby
line = "Today the weather is <%= weather %>"
weather_conditions = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_conditions[weather] = ERB.new(line).result
end
```

This endeavor, however, resulted in the following error:

```bash
NameError - undefined local variable or method `weather' for main:Object:
```

After some searching about on the internet, it appears that there is an ever-present `binding` variable, that is an instance of the `Binding` class. Strange as that was, it seems to be needed by `ERB.new` to properly interpolate variables.

The final code:

```ruby
line = "Today the weather is <%= weather %>"
weather_conditions = {}

["warm", "cloudy", "windy"].each do |weather|
  weather_conditions[weather] = ERB.new(line).result(binding)
end
```
