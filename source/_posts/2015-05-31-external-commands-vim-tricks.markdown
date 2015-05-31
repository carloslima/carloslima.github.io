---
layout: post
title: "External commands - Vim Tricks"
date: 2015-05-31 13:37
comments: true
categories: vim tricks
---

While editing code, I frequently cycle between changing the code and running some commands: running
unit tests, linting tools, style checker, restarting services, etc.

I generaly do that directly from inside vim with `:!command` and then repeating it with `:!(up arrow)`

While this is good, I learned a trick a while ago that still ranks as one of my favorites.

# The trick

Add this to your `.vimrc`

```
map \] :wa<bar><UP><CR>
```

Then run your command as `:wa|!command` and repeat it with just `\]` directly on normal mode.

If `\` and `]` are not next to each other on your keyboard layout, just change the mapping
to something that is very easy and convenient to type :)

# The advantage

This is quicker to type, it saves all the files before running the command and it still allows you
to run that one-off `:!command` in between your cycle without it getting in the way of your repetition.

That's it.
Happy Vimming!
