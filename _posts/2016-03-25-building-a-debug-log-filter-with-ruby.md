---
layout: post
title: Building a Debug Log Filter with Ruby
description:
tags: [debug,ruby,rails]
comments: true
---

> This post follows the same style as my first two blog posts, where I pick up 
a story and explain all the aspects needed to obtain a final product, with as
much detail as possible.

Logs are one of the most trustful tools you can use in any application
environment: development, test and production. They are usually the first
**and** the last thing that you can resort to in order to debug any issue, and
they cope very well with the nature of complex systems. One key factor of a
good application is proper logging, so you should always keep that in mind
while developing your applications.

Logs are also incredible tools to use _while developing_, though lots of
developers don't use them very often. They usually retain a ton of information,
but you can use plain text tools to find out what you need within them. This
tip focuses on debugging your app and filtering output right away, by means of
your log file.

## Monitoring our log file

We can use the `tail` command to monitor our log file:

```sh
tail -f logs/development.log
```

This command reads the last 10 lines of a file by default, so it's very fast
and effective _even with gigantic files_.

The `-f` flag tells `tail` not to stop when the end of the stream gets reached,
and to always keep listening for more. So the final effect is that it will
first print out the last 10 lines of the log file, and then keep printing out
**everything** that gets appended to the end of the log file – all of this in
real time.

## About text processing scripts in Ruby

We will use Ruby to build a quick one-off script that ends up acting as a log
filter.  You may be asking now: can't I just tail the log file, or open it up
and grep what I need?  Yes, but that's not what I call a frictionless
methodology.

Ruby is a fantastic tool for text processing. It actually _wants_ to be used
for that, and it even has some special flags built-in the interpreter to make
our scripts short and sweet. For example, if we start up our script with `ruby
-n script.rb`, Ruby will automatically assume that all our code is wrapped
within the following loop:

```ruby
while $_ = STDIN.gets
  # your code...
end
```

Here's one of the simplest script that we can build by using this tecnique:

```sh
echo "line 1\nline2" | ruby -n -e 'puts $_'
```

This command simply: outputs "line 1" and "line 2" to standard output; uses the
`-e` flag to run some inline Ruby code; uses the `-n` flag to iterate through
standard input. The `$_` variable represents each standard input line. We can
also shorten the `ruby` call to `ruby -ne`.

## Do we really need to build a script?

You may be asking now: why do I need to build a dedicated script for such a
simple task? I could do the same with `tail` and `grep`! Here's how:

```
tail -f logs/development.log | grep DEBUG
```

The problem is not _that_ simple. For that to work you will have to prefix
every debug line with `DEBUG`, and even map your debug strings to add `DEBUG`
in front of all their lines.

The heart of the problem lies in filtering a range of lines, so that our script
meets all possible needs within what it's supposed to do.

## Collecting requirements for our script

A log file can be huge. Clarity is very much appreciated while debugging, so we
want the cleanest output possible.  Our script will be piped from the `tail -f`
command, effectively acting as a real time filter. We will write debug
statements in our app files and be able to watch them right away. Of course,
we'll also have to run our application code to trigger that out.

So here's our desired kind of output:

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

How might we do that? We'll have to output two special kinds of lines to our
log file: one to mark the start of our debugging lines, and another one to mark
the end of our debugging lines. So here's how we might do that in our Ruby app,
assuming that our logger object is assigned to a global `$logger` variable:

```ruby
$logger.debug '#DEBUGSTART'
$logger.debug variable_1.inspect
$logger.debug variable_2.inspect
$logger.debug '#DEBUGEND'
```

Of course, we'll also have to code our log filter script in order to filter
that out.

## Creating a snippet

We can create a snippet in our editor to smooth out our workflow, which shall
be useful whenever we want to start a debugging session.  I personally use the
**vim** editor and the
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

I just type **dlog\<TAB\>** and my editor outputs the snippet to the current
buffer, thus putting the cursor right where I need it to start off my debugging
session. 

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

Notice that this script will automatically run with Ruby's `-n` flag. That flag
gets even more useful together with the `-e` flag, because we want our scripts
to be as short as possible when running inline Ruby code on the command line.
Nevertheless, we'll keep the `-n` flag here for educational purposes, although
we could pack the script up within a glorious one liner.

There are some unusual pieces in this script which are worth a quick
explanation.

The **BEGIN** method is used to initialize values in our script. it only gets
evaluated once, regardless of how many times it gets called. It receives a
block containing the values to be initialized, as well as any kind of startup
logic.  If we didn't use the `BEGIN` block, the variables would get initialized
over and over again, after every `STDIN` loop iteration. Note that there also
exists an analogous `END` method.

Second, there is a strange kind of range on the first `if` statement – what is
that? That is an obscure Ruby feature called the **flip-flop** operator. It
sure looks like a regular Ruby range object, but it is not.

Here's how it works: our **flip-flop** expression starts out set to `false`,
and will be first evaluated to `true` when the left side of the expression (the
"flip") first evaluates to `true`.  After that, our **flip-flop** expression
will remain `true` _until_ the right side of the expression (the "flop")
evaluates from `false` to `true` – after which our whole **flip-flop**
expression will evaluate to `false` again.  In our particular case that means
we are outputting all the lines that exist between two delimiter lines,
including the delimiters themselves.

Our script is also prettifying the delimiter lines, by taking out the ugly
`#DEBUGSTART` and `#DEBUGEND` strings and replacing them with hashbangs only.

Thirdly, we are using a **case statement** in order to match our delimiter
regexes.  The **case statement** is a wonderful and kind of unique Ruby feature
– it works behind the scenes by using the `===` method (case equality method).
We will not get to much detail, but the `String` class implements the case
equality method, which is also used internally to match regexes. So what we did
is exactly the same as:

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

It did seem like a lot, but it really isn't. We are getting into the details,
that's why. So here's how our workflow might look like in the end (suppose that
we saved our log filter script to an executable file called `filterlogs`, which
is accessible throughout our path):

1. Start listening to the logs.

    ```
    tail -f logs/develoment.log | filterlogs
    ```

2. Fire up your snippet to start debugging. Inspect any variable or value that
   you want.

3. Run the code

4. See the clean debug output.

## What about pry?

**Pry** is an awesome tool, and it sure belongs to my toolbelt. That said, i
just bother to fire it up when the current debugging method gets exhausted and
I don't know where to start looking up things. I also use **pry** as a debugger
for more complex scenarios, but the current method satisfies just about 90% of
my needs.

## Bonus 1

It might be useful to output some log context before our debugging lines.  The
idea here is that we run the following command to include context before our
main output:

```sh
# Show 20 lines of log context before our debug lines
tail -f log/development.log | LOC=20 filterlogs
```

Here's our final script with that feature:

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

This technique gets really powerful when we use a terminal window with split
panes, specially when we have the log filter running right next to our editor.
**tmux** is a great terminal multiplexer tool, but we can do just fine with
**iTerm 2** as well.

Can we use this technique while testing? Absolutely. The benefits are that our
debugging output does not get mixed in between our main test output. It's
really great not to have to scroll up to see what you want, nor look for some
lost output in the middle of the main test output.

## Conclusion

This post was a bit lengthy because I went on explaining everything along the
way. Hopefully there is something that you didn't know here. I hope this story
inspires you to improve your workflow, by building your own tools using Ruby's
raw text processing power.  Thanks for reading along!
