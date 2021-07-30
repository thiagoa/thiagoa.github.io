---
layout: post
title: Command Line History Tricks [en]
description: Avoid retyping and rework
tags: [shell bash zsh]
comments: true
---

You can read the brazilian portuguese version of this post [here](http://thiagoa.github.io/dicas-de-historico-de-linha-de-comando/).

There are usually shortcuts for any given computer task to make it go faster or be more precise. However, the problem is not being slow; the point is, dragging a cursor to a target one character at a time (holding down an arrow key or something), copying and pasting with the mouse, etc, quickly gets tiring and unexciting. It builds up mental overhead over the course of a day.

When it comes to the shell, there are multiple ways of being precise. Shells have advanced history expansion and editing capabilities. I don't mean to overwhelm you with this post. The goal here is to provide you with a few shortcuts and a mental model for you to remember them, because, otherwise, they wouldn't be so easy to remember.

## The shell expansion character

The shell expansion character is `!` (bang). That is the first thing you should remember, **ever**. With a bang, you can expand:

- Anything from the current command: the full string, a specific argument, a range of arguments, etc.
- Anything from the last command: the full string, a specific argument, a range of arguments, etc.
- Anything from any previous command!

## A general example

First, I want to give you a general idea of how shell history features can be used productively.

To reference arguments from the current line, you can resort to the `!#:N` shorthand. For example, say you want to rename a file:

```sh
mv some/long/path/name !#:1
```

After typing `!#:1` and then hitting `TAB`, your command line will expand to:

```sh
mv some/long/path/name some/long/path/name
```

> TAB is just for editing the contents or for accessibility. The shell will automatically expand wildcards upon executing the command.

Now you can edit that argument to your liking, which is usually faster than copy-paste or typing it out from scratch.

Note that I expanded argument 1. Arguments start from 0, so `#!:0` repeats the first argument, `#!:1` repeats the second, and so on.

`!#:N` may be cryptic and hard to remember, but I made up a trick to help me remember the "current line" case: keep in mind that hashtags can be used to reference numbers - like #0, #1, #2, etc. -, and the shell history expansion character is `!` (bang). So, to reference the first argument, type `!#1` and then put a colon between `#` and `1`.

In the same way, you can grab the entire previous command with `!!` (double bang). So, if you forgot to `sudo` something, you can do:

```sh
systemctl stop gdm # Ooops...
sudo !! # Now this is what you want!
```

As expected, you can also get individual arguments from the last line with `!!:N`. For instance, if you want their last argument, use a dollar sign - `!!:$` - which is super helpful when typing out a sequence of commands where you repeatedly reference the last argument:

```sh
mkdir -p some/dir
cd !!:$ # I will explain later how you can simplify this...
```

## Expanding any command: Event Designators

It is possible to reference or repeat any command, including the current line:

- `!#` - Current line
- `!!` or `!-1` - Previous command.
- `!-2` - Penultimate command
- `!-3` - Antepenultimate command
- `!-N` - Nth previous command
- `!pattern` - The previous command starting with `pattern`

Let's call the above "Event Designators". Event designators specify which history line to use, and can be operated upon to extract words or ranges, which can be further manipulated (we will not touch modifiers in this article) into something else. Start with the above designators to slice any part of any command. However, they alone will expand the full command line! So if you want to repeat everything you've typed so far:

```sh
ls some/path/here !# 
```

Which will expand to:

```sh
ls some/path/here ls some/path/here
```

Not so useful, right? Expanding the full command is usually more useful to reference _previous_ commands, so keep that in mind! `sudo !!` is a good example of that.

Of course, you can use Event Designators to repeat any previous command in full:

```sh
!-2
```

Which is a substitute for pressing `ctrl+p` or `up` twice to get to the penultimate command.

## Expanding specific words

Here `:` (colon) is the "pick word operator". If you want to pick argument N, start with an event designator and suffix it with `:N`. It goes like this:

- `!#:N` - Argument N of the current line
- `!!:N` - Argument N of the previous command
- `!-2:N` - Argument N of the penultimate command
- `!-3:N` - Argument N of the antepenultimate command
- `!pattern:N` - Argument N of the previous command starting with `pattern`

Where N is an integer starting from 0. Right? The pattern here is:

```
event_designator:N
```

## Getting word ranges

Ranges are quite similar. Let's have `:` change hats and become the "slice operator":

- `!#:A-B` - Arguments A to B of the current line
- `!!:A-B` - Arguments A to B of the previous command
- `!-2:A-B` - Arguments A to B of the penultimate command
- `!-3:A-B` - Arguments A to B of the antepenultimate command
- `!pattern:A-B` - Argument A to B of the previous command that starts with `pattern`

Where A and B are integers starting from 0. The general pattern is:

```
event_designator:A-B
```

There are special word designators to exempt you from counting words, which can be used in place of A or B:

- `^` to reference the first argument
- `$` for the last argument

Here's an example:

```sh
co some/file another/file # Ooops.. it should have been cp...
cp !!:^-$ # Now it's fixed!
```

The `^-$` range is so special that there's a shorthand for it, which is `*`. For example, if you want to get all arguments from the penultimate command, use `!-1*`:

```sh
co some/file another/file # Ooops.. it should have been cp...
mkdir another/dir # No worries, I will continue doing other things
cp !-1* # Now it's fixed!
```

This may not be so real-world because modern shells will usually suggest the `cp` command for you, but the idea is to be precise when you need it!

See [word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html) for more details.

## The last word: the most common case

As I said before, `!$` represents the last argument of the last command. It is a shorthand for `!!$`, which in turn is a shorthand for `!!:$`.

In the same way, you can do:

- `!#$` - Last argument of the current command
- `!!$`, or `!$` - Last argument of the previous command
- `!-2$` - Last argument of the penultimate command
- `!-3$` - Last argument of the antepenultimate command
- `!pattern$` - Last argument of the previous command that starts with `pattern`

Can you see a pattern emerging?

> BONUS TIP: Hit `alt+.` as a shortcut to expand `!$`. Since this is the most common case, there's a shortcut for it.

## Conclusion

You can incorporate these shortcuts in your day-to-day to up your terminal game! Having a mental model is very important because, otherwise, they will seem super cryptic and you will not be inclined to use them, but they do follow a consistent pattern that you can learn and practice. I hope you've found this post useful!
