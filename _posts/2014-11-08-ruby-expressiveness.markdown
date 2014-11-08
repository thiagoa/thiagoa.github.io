---
layout: post
title: With or without parenthesis?
description: Some of my personal guidelines on writing expressive Ruby code.
tags: [ruby]
comments: true
---

Ruby is one of the most expressive languages that I ever worked with. What's
funnier about that is the paradox: it can be as beaultiful or as cryptic as you
want. It's not a foolproof language either, nor does it try to protect the
developer from his own mistakes - it's a sharp tool!

Ruby is a multi-paradigm programming language; it lets you write code in many
different styles: procedural, object oriented, functional, metaprogramming and
others.  You can write cryptic Ruby code if you want, as it still has some Perl
and other obscure inheritances, but I prefer to avoid that style whenever
possible.  The **Ruby way** says we should have more than one way to write the
same code, and that's a good thing! Unless you are writing a quick command line
hack, you'll most likely want your code to be clear and expressive.

Mostly everything in Ruby is a method call, be it implicit or explicit, except
for some few language constructs. You may know that Ruby allows you to make
method calls without parenthesis.  What's the point of that?
**Expressiveness**, of course!

First of all, I read the line out loud: does it better express the intent *with*
or *without* parenthesis?

## With the parenthesis

Take that piece of code:

{% highlight ruby %}
def update_names(additional_names)
  @names.update additional_names
end
{% endhighlight %}

Read the body out loud: it reads like plain english, and the lack of
parenthesis reinforces that feeling. At first glance it seems like it's telling
"hey @names, update the additional names!"...  so, will our names add something
to the **additional\_names** collection?  It doesn't make sense! Of course we
know what this line is supposed to do, but the real intent would be clearer at
first glance if we could write it like this:

{% highlight ruby %}
def update_names(additional_names)
  @names.update with: additional_names
end
{% endhighlight %}

Now let's add the parenthesis:

{% highlight ruby %}
def update_names(additional_names)
  @names.update(additional_names)
end
{% endhighlight %}

The difference here is that the parenthesis makes it clear that we are working
with an *expression*, a mechanical method call that is not supposed to read
like english, reinforcing the idea that **additional\_names** are being added
to **names**, and not the other way around. The bottom line is that the english
sentence gets confusing without the parenthesis.

The **update** method is part of the Ruby **Hash** class, and it's an alias for
the **merge!** method.  You will find that kind of thing in Ruby a lot. The
language makes it very easy to alias methods, and encourages you to do so.
There are many methods with different names which do the same thing.  For
example, take **map** and **collect**: both methods iterate through an array,
transform its values into something else, and return a new array at the end.
The point of having both is that there are situations where **collect** is more
readable, and others where **map** wins. **Intent** is the word.  We could go
on with a big list: **inject** and **reduce**, **detect** and **find**, etc.

Another situation where I prefer parenthesis is when the return value is
meaningful, because it's clear something will be returned to the caller:

{% highlight ruby %}
def miniaturize_adult(adult)
  to_dwarf(adult)
end
{% endhighlight %}

Now let's take a look at another example:

{% highlight ruby %}
prices = items.collect(&:price)
{% endhighlight %}

There is also a return value here, so it reads really well. Even if we tried to
run it *without* the parenthesis we would get a warning, because the &
character would confuse the Ruby parser. Ruby tries to make an assumption
about what you want, but it isn't entirely sure whether you meant to use an
overloaded operator or the **Symbol#to\_proc** syntax. It tries to be useful,
pointing out for potential errors in your code.  Note that we need to run the
Ruby interpreter with the *-w* flag to see that warning.

## Without the parenthesis

Method calls written *without* the parenthesis are essentially *imperative*
calls where the return value *doesn't* matter. For example:

{% highlight ruby %}
# Example one
puts "That's an order, just do it!"

# Example two
assert_equal 'hello world', output

# Example three
fill_in 'Name', with: 'Thiago'
{% endhighlight %}

Of course, there are some situations where we can't do that, and for a good
reason.  Ruby emits a warning if you try to use the following syntax:

{% highlight ruby %}
assert_matches /world/, output
{% endhighlight %}

For the same reason described above, that line confuses the Ruby parser: who
knows, you could be trying to perform a division between two variables. So we
are obliged to include a parenthesis in that method call:

{% highlight ruby %}
assert_matches(/world/, output)
{% endhighlight %}

## Conclusion

Everybody has its own guidelines when it comes to expressiveness, but the style
presented here aims to be consistent. Some people simply hate parenthesis,
taking them out whenever they can - even in method definitions, which I
personally don't like! Parenthesis in method definitions add structure and make
the code easier to read, effectivelly communicating "this is a method body".
Let's not be too radical, OK?  Parenthesis actually have a purpose!
