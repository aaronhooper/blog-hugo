---
title: "Getting faster at searching and typing"
date: 2020-06-01
draft: true
---

The following post contains some tips that I have gathered over the
years that have helped me to get things done on the computer faster, in
particular web browsing and typing/word processing. I think these tips
may be useful for journalists, programmers, researchers, or anyone who
is doing a lot of simultaneous reading and typing.

## Avoid using the mouse
The mouse is good for many things. It is intuitive and accessible, but
it becomes a burden when you want to do things fast, for two reasons:

- it is slow to move your hands to and from the keyboard home row
- it relies on your ability to accurately click what you want, which is
  harder to do fast

It is very difficult and not at all worth it to attempt to completely
remove your dependency on the mouse for doing things like browsing the
web. You can, however, limit the amount of times you have to move your
hand away from the keyboard by learning the keyboard commands for your
applications.

There are a few advantages to preferring the keyboard over the mouse:

1. The keyboard is highly mappable, and even if you have a gaming mouse
   that has 12 buttons on the side of it, there is still more room for
   expansion with the keyboard.

2. Keyboard shortcuts allow you to express exactly what you want with a
   great degree of accuracy. No more missing the close button by two
   pixels.

Every operating system has methods available to create your own
keybindings in, to do simple things like open your favourite
applications, and more complex things like upload an image to a hosting
site for sharing.  There are options to change bindings on Windows and
macOS from within their native settings, and more powerful features can
be sought out from third party programs.

Common third party applications for more advanced actions are
[AutoHotkey][1] for Windows and [Hammerspoon][2] for macOS. Linux users
will have to refer to their specific distribution, but [`sxhkd`][3] is a
common program used to configure bindings across all distributions,
especially those who choose not to use a full desktop environment.

### Tips for binding keys
I would recommend learning the keyboard bindings for your particular
applications before opting to rebind them to something else, if you have
that option. It's easy to get too excited and bind a whole bunch of keys
at once without having a real reason for changing them, and in this case
it is likely that you will quickly forget that you even set them in the
first place.

It is better to instead find and use what is already available to you,
so that you can have a good reason for wanting to change it. Many
keyboard bindings in mature software applications come from either years
of convention or what works the best for the most people. These
conventional key bindings are also likely to work for similar actions
across different applications, keeping your hard-earned muscle memory
"portable".

## Browser
The web browser is by far the most commonly used application on a
computer, and so they are filled with well-known and not so well-known
shortcuts taken from other commonly used programs.

### Search
Stop clearing your browsing history. Seriously, no one cares that you
watch porn. Also, get into the habit of only searching from the URL bar.

The reason for this is simple: if you are searching for something which
you have already visited before, then it will appear in the drop down
menu, which you can select and it will take you directly to the webpage,
or an existing tab that already has the webpage open. Going instead to
the search engine to find what you already might have in your history
**can take two or even three page requests**. Even if you have a fast
internet connection to rely on, that's a lot of jumping around!

If you are reading an article which has a lot of unfamiliar words, you
might open new tabs and type or copy-paste the words into the search so
that you can get a better idea of what you're reading. This will get
tedious very quickly.

Instead, the following things exist to help you out: double-click
selects the word under the cursor, and adding a drag before releasing
selects multiple words. This is more foolproof than having to select a
word or phrase by precisely moving the cursor to the beginning character
and dragging to the end character, when all you care about is the words.
Now, why is this useful at all?

If you right-click the selected text and press `s`, a new tab opens with
the selected text entered into your favourite search engine. Chances are
it also has a Wikipedia summary box included on the right too, saving
yet another page request, and, if you are a right hand mouse user, it
keeps your left hand on the home row.

This gives hyperlink-level convenience for searching every possible word
or phrase you could ever read, and you can very quickly get a
surface-level understanding about a complicated topic with lots of
technical jargon using these shortcuts.

If you hear a word you don't know when talking with a friend, or
listening to a podcast or TV in the background, you can type `Ctrl-T`
`define <word>` `Enter`, read the definition, then close the tab
with `Ctrl-W`. Very snazzy.

### Find on page
If you want to find other occurences of a word in the page, you can
double-click it and press `Ctrl-F-G`, which will search the page for the
selected word and find the next occurence after that, triggering
highlighting for the entire page. Note that in Firefox the highlighting
must be enabled first by clicking `Highlight All` next to the search
bar, but once selected it stays on for all future searches. This is
especially useful for programmers who are reading source code in the
browser and want to quickly find the other locations where a particular
variable is used.

Quick finding can also be done with `/`, though some websites override
this with their own search. In Firefox you can enable searching with
quick find without `/` by navigating to `Preferences`, `General`,
`Browsing`, and checking `Search for text when you start typing`. It is
an absolutely killer feature that is worth checking out. I believe an
extension is needed for the same functionality in Chrome.

### Other neat shortcuts
Some less well-known ones include:
- `Ctrl-L` / `F6` / `Alt-D` - select the URL bar
- `Alt-Left`, `Alt-Right` - go back or forward in history
- `Ctrl-Shift-T` - reopen the last closed tab, including attached
  history
- `Ctrl-Tab`, `Ctrl-Shift-Tab` - switch back and forth between tabs

## Search engine
Pick a search engine and learn all of its quirks. I use [DuckDuckGo][5]
because they claim to care about privacy, but they also have some nice
features that also make it my favourite search engine functionally
speaking.

There are [bang][4] (`!`) commands that enable you to redirect your
search to more specific sites, such as:
- `!g` - Google
- `!yt` - YouTube
- `!w` - Wikipedia
- `!a` - Amazon

This is extremely useful for quickly searching Google from within
DuckDuckGo as its search results aren't always as good as those from
Google. There are [literally tens of thousands of bangs][4] available
for DuckDuckGo, so you are bound to find some that are useful to you.

Many websites have their own home row bindings for doing common things.
`k` and `j` can be used for going up and down your Gmail inbox if you
enable it in the settings. DuckDuckGo has its own too: `/` selects the
search box, `k` and `j` move up and down the search results, and `l`
opens the selected result. These can be discovered through trial and
error or just through searching.

You can set your favourite search engine in your browser settings, which
means searching from the URL bar will always use it.

## OS

- some stuff about tiling window managers
- command line, common interface, scriptable

> Y'know, the ol' `./configure && make && sudo make install`? Just rolls
off the fingertips, doesn't it?

[1]: https://www.autohotkey.com/ (AutoHotkey)
[2]: https://www.hammerspoon.org/ (Hammerspoon)
[3]: https://github.com/baskerville/sxhkd (sxhkd)
[4]: https://duckduckgo.com/bang (DuckDuckGo bang commands)
[5]: https://duckduckgo.com/ (DuckDuckGo)
