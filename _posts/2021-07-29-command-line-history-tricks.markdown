---
layout: post
title: Command Line History Tricks [en]
description: Avoid retyping and rework
tags: [shell bash zsh]
comments: true
---

For any given computer task, there are usually shortcuts to make it faster and be more precise. However, the main problem is not being slow; the point is, dragging a cursor to a target one character at a time (holding down an arrow key or something), copying and pasting with the mouse, etc, quickly gets tiring and unexciting. It builds up mental overhead over the course of a day.

When it comes to the shell, there are multiple ways of being precise. Shells have advanced history expansion and editing capabilities. I don't mean to overwhelm you with this post. The goal here is to provide you with some shortcuts and a mental model for you to remember them, because, otherwise, they are not so easy to remember.

## The shell expansion character

The shell expansion character is `!` (bang). This is the first thing you should remember, **ever**. Starting with a bang, you can expand:

- Anything from the current command: the full string, a specific argument, a range of arguments, etc.
- Anything from the last command: the full string, a specific argument, a range of arguments, etc.
- Anything from any previous command!

## A general example

To reference arguments from the current command, use the `!#:N` shortcut.

For example, say you want to rename a file:

```sh
mv some/long/path/name !#:1
```

After typing `!#:1` and then hitting `TAB`, your command line will expand to:

```sh
mv some/long/path/name some/long/path/name
```

> TAB is just for accessibility; the shell will automatically expand wildcards upon executing the command.

Now you can edit that argument to your liking, which may be quicker than copy-paste or typing it out from scratch. Note that I chose to expand argument 1. Arguments start from 0, so `#!:0` repeats the first argument, `#!:1` repeats the second, and so on.

`!#:N` may be cryptic and hard to remember, but there's a mind trick. Just remember that hashtag is usually used to reference numbers, like #0, #1, #2, etc, and the shell history expansion character is usually `!` (bang). So to reference the first argument, you would type `!#1` and then put a colon between `#` and `1`.

In the same way, you can grab the full previous command with `!!` (double bang). If you forgot to `sudo` the previous command, you can do:

```sh
systemctl stop gdm # Ooops...
sudo !! # Now this is what you want!
```

As expected, you can also get individual arguments of the last command with `!!:N`. If you want the last argument, use a dollar sign, like this: `!!:$`. This is super helpful when typing out a sequence of commands where you repeatedly reference the last argument:

```sh
mkdir -p some/dir
cd !!:$ # I will explain later how you can simplify this...
```

## Expanding any command: Event Designators

There's an easy pattern to follow:

- `!#` - Current command
- `!!` or `!-1` - Last command.
- `!-2` - Penultimate command
- `!-3` - Antepenultimate command
- `!-N` - N previous command

Let's call the above "event designators". Event designators specify which history line to use, and can be operated upon to extract words or ranges, which can be further modified (we will not touch modifiers in this article). Start with the above to slice any part of any command. However, they alone will expand everything! So if you want to repeat everything you've typed so far, use:

```sh
some command line !#
```

Which will expand to:

```sh
some command line some command line
```

Not so useful, right? Expanding the full command is usually more useful to reference _previous_ commands, so keep that in mind! `sudo !!` is a good example.

Of course, you can use that to repeat previous commands in full:

```sh
!-2
```

This is a substitute for pressing `ctrl+p`/`up` two times and then `return` to repeat the penultimate command.


## Expanding specific words

Let's say `:` (colon) is the "pick word operator". "Word" is bash's terminology, but "argument" is easier to undertand. If you want to pick argument N, start with an event designator and put a trailing `:N` in it. It goes like this:

- `!#:N` - Argument N of the current command
- `!!:N` - Argument N of the last command
- `!-2:N` - Argument N of the penultimate command
- `!-3:N` - Argument N of the antepenultimate command

Where N is an integer starting from 0. Right?

## Getting argument ranges

Ranges are similar. Let's have `:` change hats and become the "slice operator":

- `!#:A-B` - Arguments A to B of the current command
- `!!:A-B` - Arguments A to B of the last command
- `!-2:A-B` - Arguments A to B of the penultimate command
- `!-3:A-B` - Arguments A to B of the antepenultimate command

Where A and B are integers starting from 0. There are special word designators to exempt you from counting, which can be used as A or B:

- `^` to reference the first argument
- `$` for the last argument

Here's an example:

```sh
co some/file another/file # Ooops.. it should have been cp...
cp !!:^-$ # Now it's fixed!
```

And there's a shortcut for that, `!!*`:

```sh
co some/file another/file # Ooops.. it should have been cp...
cp !!* # Now it's fixed!
```

This is not so real-world because modern shells will usually suggest the `cp` command for you, but the idea is to be precise when you need it!

See [word designators](https://www.gnu.org/software/bash/manual/html_node/Word-Designators.html) for more details.

## The last word: the most common case

As I said before, `!$` is the last argument of the last command line. It is a shortcut to `!!$`, which in turn is a shortcut to `!!:$`.

In the same way, you can do:

- `!#$` - Last argument of the current command line
- `!-2$` - Last argument of the penultimate command line
- `!-3$` - Last argument of the antepenultimate command line

Can you see a pattern emerging?

> BONUS TIP: Hit `alt+.` as a shortcut to expand `!$`. Since this is the most common case, there's a shortcut for it.

## Conclusion

Use these shortcuts to up your terminal game! Having a mental trick is very important because otherwise they will seem super cryptic, but they follow a consistent pattern that you can learn and practice. I hope you've found this post useful!
