---
layout: post
title: Prefer an Implicit Block
description: Why prefer implicit blocks in Ruby, and reasons to use explicit blocks.
tags: [ruby]
comments: true
---

This is an old topic, but I often see people converting blocks to procs in
method arguments, when there is no need to:

{% highlight ruby %}
# Don't do this!
def each(&block)
  @collection.each do |element|
    block.call element
  end
end

# Instead do this:
def each
  @collection.each do |element|
    yield element
  end
end
{% endhighlight %}

There is a performance hit in the first example, because we are converting a
block to a Proc object behind the scenes. Let's have a look:

{% highlight ruby %}
require 'benchmark'

Car = Struct.new(:name)

class ParkingLot
  def initialize
    @cars = []
  end
  
  def add_car(car)
    @cars.push car
    self
  end
  
  def each_car_with_explicit_block(&block)
    @cars.each { |car| block.call(car) }
  end
  
  def each_car_with_implicit_block
    @cars.each { |car| yield car }
  end
end

parking_lot = ParkingLot.new

10.times do |i|
  parking_lot.add_car Car.new("Car nº #{i}")
end

n = 5000000

Benchmark.bm do |x|
  x.report "With an explicit block" do
    n.times do
      parking_lot.each_car_with_explicit_block {}
    end
  end
  
  x.report "With an implicit block" do
    n.times do
      parking_lot.each_car_with_implicit_block {}
    end
  end
end
{% endhighlight %}

Now to the results:

    user     system      total        real
    With an explicit block 15.010000   0.160000  15.170000 ( 15.598784)
    With an implicit block  6.200000   0.040000   6.240000 (  6.361095)

We can see here that an implicit block is **more than twice faster**. It is
idiomatic Ruby, and readability isn't very compromised. Some people prefer to
have blocks at the paremeter list, but if you use short methods this is not
a problem, as you can easily see the **yield** keyword inside the method.

You will also know that the method needs a block, because it will throw a **no
block given (yield) (LocalJumpError)**, if there are cars in the collection and
you don't supply a block.

## When to use an explicit block

There are some legitimate reasons to prefer explicit blocks.

### You want to store the block for later use

A snippet of code is worth a thousand words:

{% highlight ruby %}
def on_initialize(&block)
  @on_initialize = block
end
{% endhighlight %}

### You don't want to duplicate knowledge about yielded arguments

You may want to convert a block into a Proc when delegating it to another method,
specifically when you don't want your proxy method to have knowledge about
yielded arguments. Let's have a look at a bad example:

{% highlight ruby %}
# The Renderer class accepts an optional block in its
# constructor, so it can yield itself back to the caller
def render
  # Do some setup here...
  Renderer.new(setup) { |renderer| yield renderer }.render
end
{% endhighlight %}

This is a bad example, because whenever we change the **Renderer** block
parameters we'll also need to modify the proxy **render** method, to reflect
changes in the argument list.

Instead we turn our block into a Proc object, then turn the Proc back to a
block when delegating it to the **Renderer** constructor:

{% highlight ruby %}
def render(&block)
  # Do some setup here...
  Renderer.new(setup, &block).render
end
{% endhighlight %}

The main advantage in that aproach is that the **render** method doesn't need
to know anything about the block arguments - it's also easier to read and easier
to reason about.

If you don't supply a block to the **render** method, Ruby will assume the block
argument is **nil**: when it delegates the block to the **Renderer** constructor,
it does a **Renderer.new(setup, &nil)** behind the scenes, and the constructor
will assume no block is given.

### You want flexibility

Maybe you want to do some crazy metaprogramming, inspecting your block from the
inside out; maybe you want to know the block's source location, or use a
curried version of the block... or maybe you want check its arity, or
whatever... you can still convert your block to a Proc object.

### You want explicitness in the method arguments

If you want explicitness, just use Python. OK, I'm just kidding :)

## Conclusion

Sometimes when you learn a coding technique you tend to abuse it without thinking.
Let's not do that. If you have any other reason why using an explicit block is worth,
leave it in the comments.
