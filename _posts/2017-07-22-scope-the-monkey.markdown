---
layout: post
comments: true
title:  "Scope the Monkey"
date:   2017-07-22 00:00:00 -0500
categories: ruby 
---
![monkey patch](/assets/images/monkey-patch.jpg)
This post is about limiting the scope of "monkey patches" with Ruby's Refinements.
This is certainly nothing new - there are numerous blogs posts and talks on this topic
if you just Google around a bit.  Despite Refinements having been around in Ruby since
version 2.0, it's not something we see very often.  For that reason, it can help revist
the topic in order to get re-acquainted with it, or learn something new.

## The Problem with Ruby's Open Classes
As usual, to understand a thing, we need to understand the problem that thing is 
meant to solve.

# Open Classes
Open Classes, more commonly known as 'monkey patching', is a meta programming technique 
that lets the developer add new methods (or change existing ones) to a class at run-time.
For example:

{% highlight ruby %}
class String
  def randomize
    self.chars.shuffle.join
  end
end
{% endhighlight %}
In this example, the `String` class is re-opened and the `randomize` method is added.
Doing this affects every existing instance of `String` and all new instances going forward.
This works because instance methods are stored in the class object.

{% highlight ruby %}
str = 'monkey'

begin
str.scramble 
rescue => e
  puts e.message
  #=> undefined method `scramble' for "monkey":String
end

class String
  def scramble
    self.chars.shuffle.join
  end
end

puts str.scramble 
#=> omknye
{% endhighlight %}

# So, what's the problem with this?
"Monkey patching" is global. The above change affects every `String` object in the
entire application. Some things to consider:
* Will it conflict with a 3rd party library?
* Could we accidentally override an existing method?
* Will our patch remain compatible with future versions of Ruby?

## Refine
We can limit the scope of our "monkey patches" by calling `refine` inside a module
definition:

{% highlight ruby %}
module StringExtensions
  refine String do
    def scramble
      self.chars.shuffle.join
    end
  end
end
{% endhighlight %}

A Refinement is not active just by defining it.  To activate a Refinement, it must be done
explicitly with `using`.
## Using
To activate Refinement call `using`:
{% highlight ruby %}
using StringExtensions
{% endhighlight %}

We can do this inside a module or class so that our patch is only active inside a module
or class definition.

{% highlight ruby %}
class Scramble
  using StringExtensions
  def self.call(word)
    word.scramble
  end
end

Scramble.call('monkey')
#=> knmeyo
{% endhighlight %}

A Refinement is active is two places:
1. Inside the `refine` block itself
2. Starting at the place in the code where `using` was called until the end of the definition if inside a module or
class.

## Some Gotchas
* Methods already called in a definition are not Refined after calling `using`.  Here is an example:
{% highlight ruby linenos %}
class Calculate
  def add1_to(num)
    num + one
  end

  def one
    1
  end
end

module CalculateExtensions
  refine Calculate do
    def one
      1.0
    end
  end
end

using CalculateExtensions
Calculate.new.one #=> 1.0
Calculate.new.add1_to(2) #=> 3
{% endhighlight %}
After we call `using`, it's reasonable that one would expect `Calculate.new.add1_to(2)` to return `3.0` 
due to coercion, but we're actually still adding `1` and not `1.0`.  That's because the call to `one` inside 
`add1_to(num)` method on line 3 happens before the call to `using` on line 19.

* calling `using` directly in IRB at the moment doesn't work.  You can learn more about 
this [here](https://bugs.ruby-lang.org/issues/9580).

Refinements are an easy way to limit the scope of our "monkey patches", thereby making them much
safer to implement. We can avoid unexpected results that can come with making global changes to our code, 
yet still take advantage of Open Classes.
## Resources 
* [Ruby docs on Refinements](https://ruby-doc.org/core-2.4.1/doc/syntax/refinements_rdoc.html)
