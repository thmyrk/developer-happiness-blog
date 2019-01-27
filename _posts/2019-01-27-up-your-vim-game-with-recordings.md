---
layout: post
title:  "Up your vim game with recordings"
date:   2019-01-27 21:30:00 +0100
background: '/img/posts/up-your-vim-game-with-recordings.jpg'
image: '/img/posts/up-your-vim-game-with-recordings.jpg'
categories: tools
comments: true
---

When you see my phone you immediately know which is my favourite text editor.

<div class="center-image">
  <img src="/img/posts/up-your-vim-game-with-recordings/phone.jpg" alt="phone">
</div>
_(sorry for bad quality)_

I've been using it for **years** now and due to its vast number of features, I'm constantly learning something new.
I'm not even sure whether you can say that you really **know** vim, no matter how much you study it.

Today I want to share with you the feature which I always neglected though I always knew of its existence and the one
which improved my productivity when forced to do some tedious editing.

# VIM recordings

VIM recording, as the name suggests, records everything you type.
To start recording, press `q` in normal mode followed by a letter (a to z). That starts recording keystrokes to the specified register.
Vim displays `recording` in the status line. Type any normal mode commands, or enter insert mode and type text.
To stop recording, again press `q` while in normal mode.

Let's see an example of how I would use this feature.

<script id="asciicast-WnQXE7EUUmtSEVaugJLdf0mBf" src="https://asciinema.org/a/WnQXE7EUUmtSEVaugJLdf0mBf.js" async></script>

# How to use it

The problem with using recordings is the fact that in order to use them successfully you have to use only the combination of commands that
will yield the same result when repeated in different lines. Let's consider the following:

You want to transform these two lines:

```
First name: John
Address: 5 Central Park West and Columbus
```

into these:

```
<dt class="property-name">First name:</dt><dd>John</dd>
<dt class="property-name">Address:</dt><dd>5 Central Park West and Columbus</dd>
```

We start on the first line with the cursor on the first character *F*. First steps are simple, we go into insert mode `i` and type in
`<dt class="property-name">`. We exit insert mode `ESC` (I have this bound to `jj` which I recommend). The cursor now is located at `>`.
Now we need to think how to move our cursor in a repeatable way to put `</dt>` after `First name:`. The simplest way would be to move 11 times
to the left using `11l`, but obviously, this is a bad choice because `Address:` has 8 characters and we would end up with our cursor after
the word in that line. A step in the better direction would be to jump by words `3w`, but `Address:` is one word where `First name` are two so
again, it's a no-go. A common characteristic of these two lines is the double-colon `:`, so the best course is to jump to this character `f:`,
go one character to the right `l`, go to insert mode using `s` and type `</dt><dd>`. Then we can just exit insert mode `jj` and start typing
at the end of the line `A` and type the closing tag `</dd>`. Next, we exit normal mode `jj`, go one line down `j` and go to the beginning of the line
`^` and finish the recording `q`. The last steps are necessary if we want to be able to repeat a recording for many lines. In
another case, the recording would repeat itself for the same line.

## Use it with commands too

<script id="asciicast-pwuGw79WV9ui401VJ3aEk3lB6" src="https://asciinema.org/a/pwuGw79WV9ui401VJ3aEk3lB6.js" async></script>

# Conclusion

For less-complicated repeatable tasks, I often use `.` or `<CTRL>-v` and other visual selections when possible.
It's not often that I use VIM recordings, but when I do I'm very grateful I'm able to use them because they can prove
to be a useful tool in certain cases. For me, it's mostly when editing HTML.

So there you go! VIM recordings are awesome!
Feel free to leave any suggestions for some use-cases below or if you feel I should add something in this post :)

Thanks for reading!

Przemysław
