---
layout: post
title: "Escaping from Ruby Blocks"
date: 2017-10-18 12:00:00 -0400
description: |
  If you need to bail out of a Ruby block before the end, how is that accomplished?
tags:
- ruby
aliases:
- /posts/2017-10-18-escaping-from-ruby-blocks
---

## Guard Clauses

Once upon a time, [Rubocop taught me to love guard clauses](http://rubocop.readthedocs.io/en/latest/cops_style/#styleguardclause). When I first started learning to program, I was deeply conflicted about guard clauses. On the one hand, guard clauses can prevent the need to have deeply nested `if`s. On the other hand, it can be hard to remember the context deep into a method without being able to reference the enveloping `if`s (similarly to RSpec contexts). In any case, Rubocop wants me to use guard clauses, so I do.

Here's an example of using a guard clause—if the author is `nil`, there's no point in continuing with this method.

```ruby
def describe_author(author)
  return "" if author.nil?
  "#{author.name} (#{author.birth_year}–#{author.death_year})"
end
```

## Blocks

The Ruby language has a useful construct to contain blocks of code, aptly called "blocks." Actually, Ruby has three ways to contain blocks of code—blocks, procs, and lambdas—but this post will just cover blocks.

## Guard Clauses in Blocks

First, in order to use a block, we'll need a function that expects one.

```ruby
def foo
  yield if block_given?
  puts "End of foo"
end
```

This method will execute whatever block we give it first (*if we gave it one*), then print out a message. Let's give it a block.

```ruby
foo { puts "Inside block!" }
#=> Inside block!
#=> End of foo
```

Now let's suppose that we want a guard clause in the block that we send.

### return

My first inclination on how to do this was with the `return` keyword. Seeing `return` in a block, though, would probably raise some questions about whether it's the block that's being returned from or the method that `yield`s to it. Let's try it and find out.

```ruby
foo { return }
#=> LocalJumpError: unexpected return
```

It doesn't return from the block or the calling method, it just crashes. Interesting!

### break

The `break` keyword was the next suggestion to come to mind. You may have seen the `break` keyword used to break out of blocks early in cases like this:

```ruby
def find_first_even(array)
  first_even = nil
  array.each do |item|
    if item.even?
      first_even = item
      break
    end
  end
  first_even
end
```

Of course, this example is completely contrived—one could simply use `Array#find` here. Regardless, when (or if) an even item in the array is found, `first_even` is set to that item, and `break` is used to stop iterating.

The [break statement documentation](http://ruby-doc.org/core-2.3.4/doc/syntax/control_expressions_rdoc.html#label-break+Statement) has this to say:

> Use break to leave a block early.

OK! Let's try using `break` in our block.

```ruby
foo { break }
#=> nil
```

What's that? Silence? Something odd has happened here. The end of the `foo` method had this line:

```ruby
puts "End of foo"
```

However, that message wasn't printed to the console. Why? 

It appears that the `break` statement in a block actually returns from the calling method. When you consider our example above, this behavior seems to make sense. `Array#each` calls the given block with each element of the array, in turn. When we call `break`, it doesn't exit from the block that `each` would then call again with the next item—it exits from the `Array#each` call.

### next

Using this insight, there's another alternative! Let's make another contrived example.

```ruby
def evens_from_array(array)
  evens = []
  array.each do |item|
    next if item.odd?
    evens << item
  end
  evens
end
```

Naturally, you could have just used `Array#select` instead. In any event, on each iteration, we skip over the rest of the block if the item is odd. This seems like what we want! Let's give it a try.

```ruby
foo { next }
#=> End of foo
```

Perfect!

Let's make a somewhat less contrived example, just to make sure that it really does work the way that we hope. Here is an (admittedly worthless) "calculation" method.

```ruby
def calculation(a, b)
  yield(a, b)
end
```

Let's use a guard method to exit from the block early.

```ruby
calculation(23, 0) do |a, b|
  next 1 if b == 0 # 23 ** 0 will always be 1.
  puts "Calculating..."
  a ** b
end
#=> 1
```

Great! The "Calculating..." message is not printed, as we exited the block prior to that point. Now let's use the same block, but call `calculation` with arguments that will not trigger the guard clause.

```ruby
calculation(23, 2) do |a, b|
  next 1 if b == 0 # 23 ** 0 will always be 1.
  puts "Calculating..."
  a ** b
end
#=> Calculating...
#=> 529
```

It works!

I hope that—in the edge case that you, too, share a preference for guard clauses *and* find yourself needing to pass some sort of large block—have found this helpful are able to put this knowledge to good use!

I don't use comments on my site, but if you have some feedback that you'd like to share, feel free to tweet at me or send me an email using the links below.
