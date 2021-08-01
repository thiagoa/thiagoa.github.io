---
layout: post
title: My Linux Setup [en]
published: false
description: There has never been a better time to be a Linux user
tags: [linux open-source]
comments: true
---

1 year and a half ago I decided to bite the bullet and go Full Linux. Before that time I've been a macOS user for 10+ years, but I became increasily dissatisfied with Macs and their software ecosystem.

All of the Macs I owned, a total of three, became unusable from 2 to 3 years. The display my last MacBook Pro had suddenly bricked, and after an expensive repair, 5 months later, the motherboard was gone as well - so my _entire computer_ got bricked again, despite all of my hope that it would last after the repair. While googling, I realized 2017 MacBooks were doomed, as a lot of people were facing serious hardware problems with them (no, my problem was not with the keyboard; I even liked it!). My past experience also showed how fragile Macs can be. Everytime I'd buy a new Mac I'd think "I've been very unlucky but this time it's going to be different". Has it ever been different?

Also, my oldest Mac, which still works to this day after a few expensive repairs, can't run the latest version of Apple's operating system. I understand their business model, but that's not what I want for my machines. I want every machine I own to last and provide value as long as it possibly can. The switch to ARM makes the matters even worse, as every Intel Mac in the world now has an expiry date.

Does it means Macs are bad and you should avoid buying them? Not at all. Maybe I was unlucky during that 10 year span and Macs are indeed very durable!

## About Linux

My curiousity for Linux aroused in 1998. I used it for 9 years non-professionally, 3 years professionally professionally, then I switched to Macs. On the Mac, I discovered productivity tricks that became part of my profesional repertoire forever, requirements for every new operating system I'd consider dipping my toes into (of course, I'm open to new ways of doing things!). Also, nowadays I'm more experienced in improving and automating my setup, and given that Linux is an extremely flexible OS, it seems like the perfect fit for me.

I've gone through many distros in the past: Slackware, Gentoo, Red Hat, Ubuntu, SUSE, etc -, and trying out new distros is less of an interest for me than it was back then. Now, all I need is a distro with a good GNOME experience and a good packaging system (bonus if it's an industry standard that's easily be googlable), and I found that distro in Pop_OS!.

Pop_OS! is based on Ubuntu, and is a better Ubuntu in my opinion. Some people say it's just a skin, but it's much more than that: aside from very nice and consistent UI additions, it heavily focuses on user productivity. It ships with GNOME plugins for automatically tiling windows, revamped configuration panels, a nice app store (a "store" where everything is free), better hardware support (especially for graphics), a focus on keyboard shortcuts (but also mouse accessibility), and so on. It inherits everything that's good from Ubuntu and adds its own set of goodies on top. I'm old enough to want a stable and flexible system that I can just start using and be productive right away.

I could live buying one Mac after another for very futile reasons, but after my new freedom experience I don't feel like ever going back, unless I need Mac-specific software that can't be used with a virtual machine.

## My main computer

My main computer is a workstation with the following configuration:

| CPU         | AMD Ryzen 3900x   |
| GPU         | Radeon Rx 5700 XT |
| Motherboard | ASUS X-570 Plus   |
| RAM         | 64 GB DDR4 ADATA  |
| SSD         | 512 GB ADATA      |

It's a very performant computer that handles everything I throw at it, and Linux works very well on it. Actually, I had one occasional issue with suspend, but after the last kernel update it seems to be gone (hopefully!). I also have a dual HiDPI monitor setup and my overall experience has been great! My keyboard and mouse (a trackball, actually) are Logitech's and they work very well with the Logitech unifying receiver.

My laptop is a Dell XPS 13 and I've yet to notice a hardware issue. I never thought a Linux laptop could be that great, and I'd even wager to compare it to Mac laptops, which, allegedgly, are the best _laptops_ you can find on the market (when it comes to "just working" - that is, until it bricks :D). I expect a good laptop to:

- Have great battery life
- Close the lid and it sleeps smoothly
- Open the lid back up, it wakes up fast
- Have a great trackpad surface + multi-touch gesture support
- Have a great keyboard
- Be portable and lightweight
- Have an operating system that works very well on it

### When Linux is not good

To be honest, I was pretty lucky this time. Graphics support for my previous Dell laptop with an NVIDIA prime GPU really sucked, and I had many problems with it. It handled one monitor well, but two monitors, not so much. Suspend had many nagging issues, and things that worked before kept breaking between driver updates. Dual monitor setup was highly unstable under that setup. Otherwise, my laptop worked great and I had no complaints.

When going with Linux, carefully research your hardware choices if you want to have a good experience. Some say nowadays Linux is plug-and-play, and I'd say it _mostly_ is; but not uncommonly, there's that thing that does not work reliably or does not work at all. And even with that, you have to be prepared to face any emerging issues. Hopefully, I did not face any this time around, so I'm pretty happy!

## Characteristics of a productive software setup

Let's have a quick look at some features that are important _for me_. To be used most effectively, a computer system should provide software that allows me to enjoy or script quick access through anything I use on my day to day. I want to cut corners and to get to the meat of what I need as fast as I can. For example:

- A quick way to launch and switch to apps
- A quick way to access files and UI configuration panels
- Flexible keyboard shortcuts to do anything: position and move windows between workspaces and monitors, switch workspaces, and so on
- Flexible mouse shortcuts
- A text expander
- Clipboard history
- Day-to-day cloud storage
- A password manager
- A quick way to look up documentation (because I'm a programmer)
- A dropdown terminal (quake style) to avoid context-switching
- Quick look up of URL bookmarks
- Quick browser tab access
- Quick way to look file contents
- A handy translator and dictionary (because I work with English on my day to day)
- Basic level of UI programmability
- Quick web searches
- Quick GitHub lookup
- Quick Heroku lookup
- Quick AWS access
- Etc.

Some of the things I listed can be done through the web browser, but without a dedicated tool or script they require **context-switching** to a browser window and then following other steps, which can be disruptive. macOS' Alfred app made me very demanding in that regard, and all productivity tweaks, as long as one commits to using them, do really pay off! In short, I want to avoid context-switching as much as possible.

Does it mean I can't use a computer without those features? Absolutely not. It's just that it's so much better with them. I'm by no means an afficcionado and I no longer spend time optimizing things that won't bring me value.

Not extensively, let's go over some of the techniques and software I use. I won't go into details, so if you want to know more about any topic please let me know and I'll write new posts to fill the gaps :)

## Dotfiles

## Quick Look

Quick Look is one of great things about macOS. It's such a simple and important idea that I wonder why other operating systems don't ship with something similar. I can certainly live without it, but I'm much better off with it.

Thankfully, [`gnome-sushi`](https://gitlab.gnome.org/GNOME/sushi) is a good Linux alternative. It works with every file I tried: documents, images, PDFs, videos, and so on. It's not perfect but it serves me well enough. Unfortunately, installation is not that easy; at the time of writing, the version available in apt does not work with GNOME 3.38 so I have a script to install it manually.

![Gnome Sushi](/assets/my-linux-setup/gnome-sushi.jpg)

In my dotfiles, I'm installing the dependencies [here](https://github.com/thiagoa/dotfiles/blob/master/linux/packages/setup.sh#L73-L85) and I have a dedicated script to clone and install it [here](https://github.com/thiagoa/dotfiles/blob/758fcda910edaae6ecacf1a6337e2e8a568fc153/linux/packages/setup_gnome_sushi.sh).

## Text editor

I use VS Code for JavaScript and Emacs for everything else. I also use Neovim for quick edits, even though I used vim exclusively for more than 10 years for programming. This is such a big topic that I'll leave to other posts!

## Launcher

Many of the topics I listed fall under the "launcher" category. On the Mac, I used to do everything with Alfred but in Linux I get the same efficiency with a slightly more fragmented setup. I use two launchers:

- Gnome Activities to launch applications, find files, UI configuration panels, and so on. Pop_OS! has a launcher feature but I don't like the user experience (in fact, it's one of the few things I don't like about it - but I will be happy to switch to it when it improves!). I also have a dedicated file finder that I scripted with `ag` and `fzf`;
- Ulauncher to script access through my system and 3rd party services.

![Ulauncher](/assets/my-linux-setup/ulauncher.jpg)

Why two launchers? Ulauncher, ironically, is not good at launching applications. It's inconsistent in that regard; sometimes it will open a new window, sometimes it will switch to an existing application window, you never know. I expect a good launcher to switch to open applications when possible, unless I explicitly tell it not to. I tried alternatives like Albert (an Alfred clone), and they seemed to work well at the surface, until I noticed serious memory and performance problems under heavy usage, which made me drop it altogether.

So, I use Ulauncher for what it's good at! It has great Python scripting capabilities, and even though it lacks a history feature, I can live with that. With Ulauncher I can do:

- Quick web searches - I can program any web searches I use on my day to day
- Quick bookmarks: I record every URL I use on my day to day as a bookmark, be it recurring meetings, specific URL services, etc, unless a full-blown plugin serves me better.
- Quick browser tab access: I use `brotab` and `ulauncher-brotab` for that.
- Quick GitHub lookup
- Quick Heroku lookup
- And the list goes on...

One disadvantage of Ulauncher is that I had to fork some plugins and tweak them to work the way I wanted, or because they no longer worked. On the flip side, Ulauncher plugins are quite straightforward to work with, and after everything's setup it's really great!

## Favorite applications: idempotent shortcuts

This is such an important feature for me, and I doubt I will find something that works better. I can switch to any application docked in GNOME (Pop OS's dock) with a single keystroke. If the application is not opened, it will instantly pop up, which is great! That's the reason why I call it "idempotent shortcuts". Also, my applications are usually configured to restore their last state when possible, so that I can have things exactly how I left them, even after reopened. A good example is Chrome.

Surprisingly, setting up "Favorite Applications" involves a set of hidden GNOME shortcuts to the first 9 docked applications. I set `Super+number` for that, so `Super+1` launches or switches to app 1, `Super+2` to app 2, and so on. Here's how you can do that:

-

Some people make scripts to launch their most used applications at certain window layouts and particular workspaces, but Favorite Applications renders that unnecessary. GNOME usually remembers window positions, so all I have to do is fire a `Super+number`, then I insist on using the same shortcuts forever, which is quicker than `Alt+tab`. Also, they work based on dock position, so I don't need to setup dedicated shortcuts for my favorite applications.

## Terminal

Old fashioned terminal windows are no longer enough for me. Terminals have to be omnipresent and just pop up (or would I say, drop down?) wherever I am, without context-switching. I use Guake for that, which is triggered by `Super+[`. Guake is very configurable and good looking, but it does not support window splits. Thankfully, that's no problem because my shell is `tmux`, which provides great window layout support. I don't even use Guake tabs because `tmux` has it all inside.

Guake also has cool features such as a keyboard shortcut to configure transparency. By default, my terminal has transparency enough so that I can refer to information behind it, but I can also toggle to a flat background when I need to focus. It also has some very nice keyboard shortcuts for lots of other tasks, like increasing or decreasing the height of the terminal window.

## Clipboard history

The native clipboard functionality in all operating systems is pretty much mediocre because no one has clipboard history. For that I use a GNOME plugin, Clipboard indicator. It has keyboard shortcuts and search works as I expect, so it's perfect for my use case.

## Text expander

I use text expanders to save me from the mental burden of forever repeating mundane text snippets such as my name, email, address, the current date and time, email templates, and so on. On the Mac I used TextExpander for a long time, and then Alfred Snippets. On Linux I use AutoKey, and it's pretty good for my needs. For example, if I want my name I just type:

```
!tas
```

And it smoothly expands to "Thiago Araújo Silva" in any application. It works as good as on the Mac. However, AutoKey is much more than that: it's a full blown automation tool and Python scripting environment. You can simulate keypresses, keybindings, scope actions per application, and perform some pretty involved wizard trickery.

The downside is that it's a technical tool that requires programming knowledge if you want to do something else other than expanding simple snippets, so it's not as easy to use as TextExpander. But it's Python, right?

## Keyboard shortcuts

I'm very well-served by what GNOME and Pop_OS! provide me. I have shortcuts to move between workspaces, maximizing windows, resizing, tiling, and dragging windows, moving windows between workspaces and monitors, opening my most used folders in Nautilus, quitting all windows of an application, toggling Crow Translate (because it's Linux, so you can hack on things, right?), and so on.

I used to remap my keyboard system-wide with [xkeysnail](https://github.com/mooz/xkeysnail) to match Emacs' keybindings as much as possible, and it worked pretty well! One day, though, I decided to drop it and never looked back. There's a great advantage in using the default keyboard shortcuts. I don't even remap Control to Caps Lock anymore! If you touch type and commit to using modifiers on both sides of the keyboard to avoid injuring particular fingers, you're well up on your way! The rule of thumb is: **never press keybindings with one hand, ever. Always use two hands**. I disliked using the home, end, page up, and page down keys, which I had replaced with Emacs shortcuts, but nowadays I enjoy using them both on laptops, small keyboards, and full-sized keyboards (when I'm not on Emacs of course). It's actually healthy to move your hands between different home positions of a full-sized keyboard, and with practice you don't need to look at the keyboard to do so. And a big bonus: I don't feel stupid when using Windows, since Windows and Linux keyboard shortcuts are pretty similar in general.

There's one exception to this rule: on the XPS 13 I had to setup Enter as Control when pressed down with [this software](https://gitlab.com/interception/linux/plugins/dual-function-keys). The reason is that it does not have a control key on the right side, and I can't live healthily without that. Otherwise, it's a great keyboard.

## Mouse shortcuts

I use the Logitech MX Ergo trackball and I love it! I know trackballs are not for everyone, but after a few months using one I don't intend to ever go back to a traditional mouse. Unfortunately, there's no Logitech options software for Linux, but there are multiple ways to configure Logitech hardware with free and open source software.

[This script](https://github.com/thiagoa/dotfiles/blob/master/bin/setup-trackball), for example, enables the use of the trackball's ball for scrolling, by holding the right button down and flicking the ball. Right button is perfect because, when held down, it has no use in any mouse I know of. It also sets some sensible acceleration defaults, which could be set in a graphical app as well.

I also use [solaar](https://pwr-solaar.github.io/Solaar/) for increased compatibility with Logitech unifying receivers and further configuration options.

My use of the mouse goes like this: I avoid it whenever possible, but I also acknowledge it is better than a keyboard at tasks like browsing the web. So here I am talking about avoiding context-switching, again: when I'm on the keyboard, I want to keep using the keyboard; when I'm on the mouse, I want to stick with it as much as possible, unless I'm faster otherwise.

And that's where `xbindkeys` comes into play. It allows me to set other functions for the same mouse buttons by combining them with keyboard modifiers. The MX Ergo has plenty of buttons, so I set a few bindings for browsing the web:

- `Super + middle click` = Closes the browser window. Middle click alone closes the browser window, but you have to point the mouse at the tab for that.
- `Super + left click` = Go back in history. This is useful for my laptop's trackpad since Chrome does not support gestures in Linux.
- `Super + right click` = Go forward in history. Also useful for my trackpad
- `Super + back click` = Go back one browser tab
- `Super + forward click` = Go forward one browser tab

Here is my [xbindkeys config](https://github.com/thiagoa/dotfiles/blob/master/xbindkeysrc)

## Password manager

I use 1Password because I never tried other alternatives since my Mac days. And you know what? Linux support is great, as good as it was on the Mac. The Chrome browser extension is great and they've recently released a desktop application that can be opened at anytime with `Ctrl+Shift+\`.

Using it is pretty simple: keep your entries organized, commit to polishing as you create them (with strong passwords), and you will be able to login to any website with a few keystrokes, effortlessly.

## Cloud storage

Here the answer is simple: I use Dropbox. I tried SpiderOak, OneDrive (through Insync), and pCloud, and all of them either eat too much CPU, syncs unreliably, or have weird file naming restrictions or rules. Dropbox is able to sync gigabytes of data in a few minutes while keeping your machine silent. I store most of my non-archived private files in it, including private OS settings that I don't want to expose in my dotfiles.

## Translator

Google translate is good, but I want my translator to open with a single keystroke without context switching, and without mixing with my browser tabs. I never found a good Alfred or Ulauncher plugin for that, so I stuck with [Crow Translate](https://crow-translate.github.io/). It's one of the few QT apps I use. My QT theme, however, is very similar to GTK's, so it feels like a first class citizen on my desktop.

I even implemented an idempotent script to toggle it and assigned the `Super+t` keybinding to it. You can find it [here](https://github.com/thiagoa/dotfiles/blob/master/bin/toggle-crow-translate).

## Email

I want to try Emacs for email someday, but I did not yet. I don't way say that's extreme before trying it out first! In the meantime, I am using [Mailspring](https://getmailspring.com/). It's a great email app, with a modern look and feel, multi-account support, and great keyboard shortcuts. It works like I expect a modern email app to work. I've tried other apps: Evolution, Thunderbird, and Geary, but discarded all of them. In general they feel old, outdated, buggy, or unreliable.

## Offline documentation browser

On the Mac I used a great application called Dash. On Linux, you don't have to pay for that functionality: there's the [Zeal](https://zealdocs.org/) app, which uses the same docsets as Dash.

## Slack and WhatsApp

The Slack desktop app is good enough, and it looks good on Pop_OS!. For WhatsApp, I created a Chrome app and I'm pretty happy with it. The webapp has good keyboard shortcut support, and that's all I need.

## Annotated screenshots and videos

For annotated screenshots I use [Flameshot](https://github.com/flameshot-org/flameshot). On the Mac you either pay for that functionality or use use Preview, which may be manual and inconvenient.

## So many other apps!

On Linux you get free and open source quality software. There's almost nothing you can do on a Mac that you can't on Linux. On the former, you usually have to pay for productivity apps, but not on the latter. And I really enjoy the consistency of GNOME and Pop OS' UI. Would I go back to a Mac? Probably not. And recently I was given the choice and I said no. I have so much freedom and flexibility that I don't feel like going back, ever. Linux is already super fast on Intel processor and it will be even more so on ARM.
