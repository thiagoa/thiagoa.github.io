---
layout: post
title: Powerful Emacs Snippets [en]
description: Showcasing the powerful idea behind mixing snippets + code
tags: [emacs ruby]
comments: true
---

You can read the brazilian portuguese version of this post [here](http://thiagoa.github.io/snippets-legais-com-o-emacs/).

One of the cool things about Emacs is that you can use actual Lisp code to expand your snippets. The purpose of this post is to show how powerful that technique can be, so that you can implement it in your own editor (Neovim is a likely candidate) if needed.

## Snippets expansion powered by code

I have two snippets for generating Ruby `class` and `module` boilerplates, powered by the [`yasnippet`](https://github.com/joaotavora/yasnippet) package. If I open `lib/job_tracker/job.rb` in my project and type in the following trigger followed by `TAB`:

```
cclas
```

It will expand into:

```ruby
module JobTracker
  class Job
    $
  end
end
```

`$` represents the final cursor position after expanding my snippet. I also have a `cmod` counterpart to generate modules all the way down. If you are a programmer, you can likely imagine how to implement this:

1. Get the file path, `lib/job_tracker/job.rb`;
2. Remove the `lib/` prefix, resulting in `job_tracker/job.rb`;
3. Remove the `.rb` extension, so as to get `job_tracker/job`;
4. Split the previous string by `/` to get a list of two strings: `['job_tracker', 'job']`;
5. Camel case the strings with the `map` function: `['JobTracker', 'Job']`. One may use a camel case library or an advanced regex for that;
6. Transform the first element into the `module JobTracker\n` string;
7. The second element would become `module Job\n` (with two spaces indentation). If dealing with the list's last string (which we are,) a class it is! `class Job\n`;
8. Close every `class` or `module` declaration with `end` at the right level of indentation.

Of course, that algorithm should be generic.

My `yasnippet` snippet for that is:

```
`(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

> $0 is a placeholder to indicate the cursor position after expanding the snippet. Note that we call a function in Lisp with `(function)`, and not `function()`, as it is usual in languages based off of C.

It is composed of three function calls, which generate the top, middle, and bottom parts of my Ruby code (there's a reason to go with three functions instead of one, and it is a limitation of `yasnippets`). `yasnippets` evaluates Lisp code surrounded by backticks and inserts the return value into the expanded snippet. Oh, we could also have literal strings within the snippet, for sure! That is actually the most common case.

If you're curious about my Lisp implementation, [it starts here](https://github.com/thiagoa/dotemacs/blob/e5448f3862b0e1e365152d72d8fbe016e753bd74/lib/lang-ruby.el#L221).

## The problem

I have to say my snippet for `cclas` was actually this:

```
# frozen_string_literal: true

`(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

It contained Ruby's `frozen_string_literal` magic comment to guarantee all string literals will be implicitly frozen by the language's runtime (to avoid mutations).

Recently, I joined a project where `frozen_string_literal` is not needed. How to tweak my snippet only for that project? I want that line of code to be expanded conditionally with a setting. Again, here's where Emacs' raw power comes into play.

## How to make my snippet configurable?

To that end I can have a Lisp function read a global boolean, and if the boolean is truthy (`t` in Lisp) then generate the `# frozen_string_literal: true` output within my snippet. It goes like this:

```lisp
(defun ruby-code-for-frozen-string-literal ()
  (when ruby-code-for-frozen-string-literal
      "# frozen_string_literal: true\n\n"))
```

And of course, I also defined a variable (setting) with `defvar`:

```lisp
(defvar ruby-code-for-frozen-string-literal t)
```

The `t` (`true`) value means that `frozen_string_literal` will be expanded by default within my snippet. How to modify that variable per project? Emacs has a feature called [Per-Directory Local Variables](https://www.gnu.org/software/emacs/manual/html_node/emacs/Directory-Variables.html). **It means I can override any `defvar` within the scope of a project**. How cool is that? Within my new project I fired this command:

```
M-x add-dir-local-variable
```

With the above command, I specified that, for the `enh-ruby-mode` major mode, I want the value of `ruby-code-for-frozen-string-literal` to be `nil`. So it generated a `.dir-locals` file in the root of my project with this:

```lisp
((enh-ruby-mode . ((ruby-code-for-frozen-string-literal . nil))))
```

That file actually existed, so Emacs just added my new setting into it.

The last step was to make a small change to my snippet:

```lisp
`(ruby-code-for-frozen-string-literal)``(ruby-code-for-fully-qualified-name-top "class")`
`(ruby-code-for-fully-qualified-name-middle)`$0
`(ruby-code-for-fully-qualified-name-bottom)`
```

I really wanted to have each function call in its own line but that would make things a bit more complicated for reasons I will not explain here.

And that is how I spent less than 5 minutes tweaking my snippet to be configurable.

## Conclusion

I have another configuration in mind for this specific snippet, which is to generate either nested or compact module definitions - but I don't need it, so I won't implement it now.

Editors with embedded programming languages are really powerful and customizable. You have almost infinite flexibility to tweak anything you want! I hope you've found this a fun read.
