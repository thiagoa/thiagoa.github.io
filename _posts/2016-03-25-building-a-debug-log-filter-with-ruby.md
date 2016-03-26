---
layout: post
title: Building a Debug Log Filter with Ruby
description:
tags: [debug,ruby,rails]
comments: true
---

Logs are one of the most trustful tools you can use in any application
environment: development, test and production. They are usually the first
**and** the last thing you can resort to in order to debug any issue, and
they cope very well with the nature of complex systems. One key factor of a
good application is proper logging.

Logs are also great tools to use _while developing_: they usually retain a ton
of hard-to-find information, but plain text tools can help with that.  This tip
focuses on debugging apps in real time using a log filter.

## Monitoring log files

We can use the `tail` command to monitor log files:

```sh
tail -f logs/development.log
```

This command reads the last 10 lines of a file by default, so it's very fast
and effective _even with gigantic files_. The `-f` flag tells `tail` not to
stop when reaching the end of the text stream, and to always keep listening for
more.

## About text processing scripts in Ruby

We will use Ruby to build a script that acts as a log filter.  You may be
asking: can't I just tail or grep the log file? Yes, but that's not what I call
a frictionless methodology.

Ruby is a fantastic text processing tool. It actually _wants_ to be used for
that, and it even has some special flags built-in the interpreter to make
scripts short and sweet. For example, if we boot our script up with `ruby -n
script.rb`, Ruby automatically assumes that all of our code is wrapped within
the following loop:

```ruby
while $_ = STDIN.gets
  # your code...
end
```

Here's one of the simplest scripts we can build using this tecnique:

```sh
echo "line 1\nline2" | ruby -n -e 'puts $_'
```

This command:

- Outputs "line 1" and "line 2" to standard output
- Uses the `-e` flag to run some inline Ruby code
- Uses the `-n` flag to iterate over each line of standard input (the echoed
text) , where `$_` represents the line.

We can also shorten `ruby -n -e` to `ruby -ne`.

## Do we really need to build a script?

I can hear you saying: why do I need to build a dedicated script for such a
simple task? I could do the same with `tail` and `grep`! Here's how:

```
tail -f logs/development.log | grep DEBUG
```

The problem is not _that_ simple. To get that working you need to prefix all
lines of the debug string with `DEBUG`. Who wants to do that?  The heart of the
problem lies in filtering a range of lines, so that our script meets all
possible needs within what it's supposed to do.

## Collecting requirements for our script

A log file can be huge, very huge. Clarity is very much appreciated while
debugging, therefore we want the cleanest output possible.  Our script will be
piped to the `tail -f` command, hence acting as a real-time filter.

So, here's the desired kind of debug output:

```
##########################################################################################
First debug iteration line 1
First debug iteration line 2
##########################################################################################

##########################################################################################
Second debug iteration line 1
Second debug iteration line 2
##########################################################################################
```

We get to that by writing two special lines in our log file: one to mark the
start of debugging lines, and another to mark the end of debugging lines.
Here's the code, assuming our logger object is assigned to a global `$logger`
variable:

```ruby
$logger.debug '#DEBUGSTART'
$logger.debug variable_1.inspect
$logger.debug variable_2.inspect
$logger.debug '#DEBUGEND'
```

Of course, we'll also have to write a log filter script to filter that out.

## Creating a snippet

We can create a snippet in our text editor to trigger the start of a debugging
session.  I personally use the **vim** editor and the
[ultisnips](https://github.com/honza/vim-snippets/tree/master/UltiSnips)
plugin, but there probably exists a good alternative for your editor of choice.
By the way,
[ultisnips](https://github.com/honza/vim-snippets/tree/master/UltiSnips) is
simply the best snippet solution for **vim**, and you should check it out if
you happen to be a **vim** user.  Here's how my snippet for debugging with
Rails looks like:

    snippet dlog "Debug with Rails logger"
    Rails.logger.debug '#DEBUGSTART'
    Rails.logger.debug $0
    Rails.logger.debug '#DEBUGEND'
    endsnippet

By typing **dlog\<TAB\>**, my editor outputs the snippet to the current buffer,
therefore putting the cursor right where I need it to inspect whatever I want.

## Our log filter script

Here's our log filter script in all of its glory:

```ruby
#!/usr/bin/env ruby -n

BEGIN {
  DEBUGSTART = /#DEBUGSTART/
  DEBUGEND = /#DEBUGEND/
}

if $_ =~ DEBUGSTART .. $_ =~ DEBUGEND
  case $_
  when DEBUGSTART
    puts '#' * 90
  when DEBUGEND
    puts '#' * 90
    puts
  else
    puts $_
  end
end
```

There are some unusual pieces in this script which are worth a quick
explanation.

The **BEGIN** block is used to initialize values. It gets evaluated once in the
script's lifetime, regardless of how many times called. If we didn't use the
`BEGIN` block, variables would get initialized over and over again, after every
`STDIN` loop iteration. And there also exists an analogous `END` method.

Second, there is a strange kind of range on the first `if` statement – it's an
obscure Ruby control flow statement called **flip-flop** operator, which will
probably resemble something you learned in college. It may look like a range,
but it's not.

Here's how it works: our **flip-flop** expression starts off set to `false`,
and is evaluated to `true` when the left side of the expression (the "flip")
first evaluates to `true`.  After that, our **flip-flop** expression remains
`true` _until_ the right side of the expression (the "flop") evaluates from
`false` to `true` – after which the whole **flip-flop** expression evaluates to
`false` again.  In our case, it means we are outputting all lines that exist
between two delimiters, including the delimiters themselves.

Our script is also prettifying the delimiter lines by taking out the ugly
`#DEBUGSTART` and `#DEBUGEND` strings and replacing them with hashbangs.

Thirdly, we are using a **case statement** to match our delimiter regexes.  The
**case statement** is a wonderful and kind of unique Ruby feature – it works
under the hood by using the `===` method (case equality method).  Without
further details, the `String` class implements the case equality method, which
is also used internally to match regexes. So what we did is exactly the same
as:

```ruby
# String#=== also implements regex matching
if $_ === DEBUSTART
  puts '#' * 90
elsif $_ === DEBUGEND
  puts '#' * 90
  puts
else
  puts $_
end
```

## Wiring it all together

So here's how our final workflow might look like in the end (supposing that we
saved our log filter script in an executable file called `filterlogs`, which is
accessible throughout the PATH):

1. Start listening to the logs.

    ```
    tail -f logs/develoment.log | filterlogs
    ```

2. Fire your snippet to start debugging. Inspect any variable or value that you
   want.

3. Run the code

4. See the clean debug output.

## What about pry?

**Pry** is an awesome tool, and it sure belongs to my toolbelt. That said, I
just bother to use it when the current debugging method gets exhausted and I
don't know where to start looking up things. I also use **pry** as a debugger
to aid in complex scenarios, but the current method satisfies just about 90% of
my needs.

## Bonus 1

It might be useful to output some context before debugging lines.  The idea
here is that we run the following command to include context before the main
output:

```sh
# Show 20 lines of log context before our debug lines
tail -f log/development.log | LOC=20 filterlogs
```

Here's how our script looks with that feature:

```ruby
#!/usr/bin/env ruby -n

BEGIN {
  DEBUGSTART = /#DEBUGSTART/
  DEBUGEND = /#DEBUGEND/

  $number_of_context_lines = (ENV.fetch('LOC', 0)).to_i
  $context = []
}

if $_ =~ DEBUGSTART .. $_ =~ DEBUGEND
  case $_
  when DEBUGSTART
    # When the debugging starts, output the context and clean
    # it up thereafter.
    puts $context
    $context= []

    puts '#' * 90
  when DEBUGEND
    puts '#' * 90
    puts
  else
    puts $_
  end
else
  # If we aren't between our debug lines, shove everything within
  # the context array.
  $context << $_

  # The context array will have at most the number of lines 
  # that we specify in the LOC env variable.
  $context.shift if $context.size > $number_of_context_lines
end
```

## Bonus 2

This technique turns out to be pretty powerful when using a terminal window
with split panes, especially when having the log filter running beside our
editor.

Can we use this technique while running tests? Absolutely. That way our debug
output does not get mixed in to test output.  It's great not having to scroll
up to see what you want, nor having to locate debug output amidst unrelated
lines.

## Wrap up

I hope this story inspires you to improve your workflow and build your own tools
using Ruby's fantastic text processing power.  Thanks for reading!
