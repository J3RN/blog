---
layout: post
title: "To Monkey Patch Ruby"
date: 2016-08-25 16:07:49 -0400
category: j3rnvsj3rn
tags:
- monkey patching
- ruby
---

This is the first of a series of posts I intend to write in a new "J3RN vs J3RN" style. Essentially, I will be stating a problem with two or more possible solutions, taking both sides in turn, and hopefully ultimately reaching a well thought out solution to the problem.

## The Problem

For my [Time Tracker application](https://github.com/J3RN/timesheet), I want to have a pretty date and time picker. However, Ruby only natively understands a handful of formats and "[American date](https://en.wikipedia.org/wiki/Date_and_time_notation_in_the_United_States)", the format used by the pretty date and time picker, is not one of them.

**The time specified in the date and picker, available as a string, needs to be converted to a `Time` instance on save, then converted back into the American date string when displayed.**

My first solution was to put two methods in a controller, `from_american_date` and `to_american_date`, which would perform these functions. However, this approach felt messy. Why should the controller know about converting time formats? What if I need this functionality in another controller?

I saw two alternatives: Monkey patching `Time` and creating a new module, `AmericanDate`.

## To Monkey Patch

If you look at Ruby's `Time` class, you will notice that it has an `iso8601` class method as well as some `rfc****` class methods whose purposes are to parse strings from those formats into `Time` instances. Additionally, each of these class methods has a corresponding instance method that converts the `Time` instance into the proper string. Thus, it would make sense to simply define a `Time.american_date` method to parse `Time` instances from American date strings and a `Time#american_date` method to convert `Time` instances into American date strings.[<sup>1</sup>](#one)

Since Ruby modules can only provide instance methods through `include`, we will need to separate the class and instance methods into separate modules, which will be added to `Time` through `extend` and `include`, respectively.

```ruby
# lib/core_extensions/time/custom_formats.rb
module CoreExtensions
  module Time
    module CustomFormats
      AMERICAN_DATE_FORMAT = "%m/%d/%Y %H:%M %p"

      module InstanceMethods
        def american_date
          strftime(AMERICAN_DATE_FORMAT)
        end
      end

      module ClassMethods
        def american_date str
          strptime(str, AMERICAN_DATE_FORMAT)
        end
      end
    end
  end
end
```

This results in the following initializer:

```ruby
# config/initializers/monkey_patching.rb
Time.extend CoreExtensions::Time::CustomFormats::ClassMethods
Time.include CoreExtensions::Time::CustomFormats::InstanceMethods
```

Why do it this way? It's elegant. It fits with the other methods in `Time` and follows Rails conventions.

## Not to Monkey Patch

Ruby gems are almost all modules. Perhaps because of this, perhaps simply because modules are cool, modules are a common way to encapsulate functionality in Ruby, even outside of gems. In fact, [this is what the `lib` directory in a Rails application is for](http://blog.codeclimate.com/blog/2012/02/07/what-code-goes-in-the-lib-directory/). In order to move our methods out of the controller, we would simply create a module in the `lib` directory.

```ruby
# lib/american_date.rb
module AmericanDate
  AMERICAN_DATE_FORMAT = "%m/%d/%Y %H:%M %p"

  def to_time str
    Time.strptime(str, AMERICAN_DATE_FORMAT).in_time_zone
  end

  def to_string time
    time.strftime(AMERICAN_DATE_FORMAT)
  end
end
```

Done! Wasn't that simple?

Why do it this way? It's easy. If I one day want to turn this into a gem (though an [American date gem](https://rubygems.org/gems/american_date) already exists), it would be pretty simple.

## Conclusion

I'm going to monkey patch `Time` and add the method to `ApplicationHelper`. While the module solution is simpler, it doesn't come near the elegance of the monkey patching solution. Hopefully, Monkey patching responsibly will help to avoid many of its pitfalls.

<a name="one"></a><sup>1</sup> The style of monkey patching I used came from [a great blog post by Justin Weiss](http://www.justinweiss.com/articles/3-ways-to-monkey-patch-without-making-a-mess/).
