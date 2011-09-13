<p><markdown>
So I have again stumbled into this situation where I relentlessly go through lists of Vim tips updating [my vim rc file](https://bitbucket.org/cheater/vimrc) as I go.

### Colours!
Probably the best thing I've found is the ability to use 256 colors (come to think of it, I've been using eight until now), which is what you get from X too. This lets you use gvim color schemes, but I'm not yet sure how this works over ssh, especially on [stupid hosts](http://www.redhat.com/rhel/versions/rhel5/ "Red Hat Enterprise Linux 5 Features and benefits").

### Sessions, persistent undo and more
Another cool thing was to find out about *:mksession* and *vim -S*, which pretty much let you quit Vim and then restore it as it was with windows, settings, etc. Use them without parameters to use the standard file name *Session.vim*.

Separately, *undodir*/*undofile*/*undoreload* are settings new to Vim 7.3 that you can use to have undo history persist between separate edits of the same file. You can find that in [my vimrc](https://bitbucket.org/cheater/vimrc).

I've also stumbled upon Vim's *quickfix* mode, which basically lets you edit a file, compile it, and then jump around to lines that the compiler mentions (e.g. errors). See *:help quickfix*.

### Some resources
I've come across a long list of reading for Vim, from [productivity tips](http://stackoverflow.com/questions/95072/what-are-your-favorite-vim-tricks), through articles that promise (and sometimes do) change your life, to people's vimrc's on bitbucket, github, etc.

Here's a few references:

1. [Top 10 Pitfalls When Switching to Vim](http://net.tutsplus.com/articles/general/top-10-pitfalls-when-switching-to-vim/) - a quick read for new users and for people considering the switch. Also, info for Mac users.

1. [Coming home to Vim](http://stevelosh.com/blog/2010/09/coming-home-to-vim/) - a good article for new and old users alike. This has some nice tips!

1. [How to use Vim ineffectively and how to get out of the streak](http://stackoverflow.com/questions/1218390/what-is-your-most-productive-shortcut-with-vim) - this one is for intermediate users. You know how to use Vim but aren't that good at it yet.

1. [A list of productivity tricks](http://stackoverflow.com/questions/95072/what-are-your-favorite-vim-tricks) - this would be very useful as some sort of "fortune" plugin that displays a single tip from this list on the blank area in a new vim window.  Alternatively, give me a spinning ascii art vim logo. This one is for advanced users.

### Using Vim for Python and other programming languages
Vim is a pretty good editor for Python. Here are some materials for integrating
it with various code development tools; Note that I have not used any of the
debuggers extensively; I don't know which are better than the others.

1. [Setting up Vim with ctags and cscope](http://cheater.posterous.com/automating-e2fsck) - scroll down to the paragraph on setting up Vim with ctags and cscope, it's really useful. Make sure to add *setlocal hidden noautowrite* to your *vimrc* for best results.

1. [Debugging Python in Vim](http://web.archive.org/web/20100704031920/http://www.petersblog.org/node/752) - sadly only through the Wayback Machine, this guide describes how to set up Vim to put breakpoints inside the Python file you are editing and run the file in the debugger.

1. [VimPdb](http://code.google.com/p/vimpdb/) - a different kind of integration of Vim with Python debuggers using Pdb.
1.  [pydbgp](http://jaredforsyth.com/blog/2010/jul/12/integrated-python-debugging-vim/) - yet another Python debugger integration, this time using dbgp. This one has even more gadgets than the others it would seem.

1. If you're for hardcore Vim scripting, [this file I stumbled upon once in IRC](http://dl.dropbox.com/u/4498886/Pastes/async_test.vim_3.html) shows you how to run a Python worker thread inside Vim with its new Python scripting abilities. This can then get any sort of updates from Vim itself, such as if a cursor moved. 

1. [A huge collection of various Vim tips](http://vim.wikia.com/wiki/Best_Vim_Tips) - also has tips for beginners and intermediate, but most of this is semi-advanced usage. Not laid out in the best way, you can scour it for things you didn't know

1. [Random tips and tricks](https://bitbucket.org/cheater/vimrc/src/tip/TIPS) that I stumble upon. This list will change.

1. [jmcantrell's vim config](https://github.com/jmcantrell/dotfiles-vim) - this guy is a very advanced user and his vimrc reflects that. You can see a lot of cool tricks in the vimrc, he also has almost everything set up under some key binding. His vimrc is laid out in a very structured way, which is probably inevitable given its sheer weight.

1. If you are using Python, consider writing [banner-style indent](http://en.wikipedia.org/wiki/Indent_style#Banner_style) instead of lisp style. I prefer it over other languages as well, for example it makes Bash look much much better. Most Python is already "banner style" because of the indentation required for the parser; the only difference is when you have "long lines", i.e. expressions that are broken up into multiple lines.

### My vimrc
Of course, no such article would be complete without the autor extolling on the virtues of [his config file for Vim](https://bitbucket.org/cheater/vimrc). I will just say that it works for Python, Bash, C, PHP and Haskell, and has a lot of cool tricks, with a lot of comments!  It is released under the GNU GPL v3, so you can use it freely and include parts or all of it in your own GPL v3 licensed config files.

### Updates
If someone puts his vimrc on BitBucket or GitHub you can additionally follow/watch the repository to see what new cool stuff the person finds. I'm following some repositories like this.

# ⁂

Additionally, I can now use Vim &amp; Co. on [all my computers](http://singularcrew.hu/vi65/). Now if I could only find the wallwart for my shaver, this neckbeard is getting unwieldy.

</markdown></p>
