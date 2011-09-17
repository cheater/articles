Title: Keyboard shortcuts and control of readline for Bash, the Python interpreter, gdb, and so on
URL slug: readline

<markdown>
I have recently been brushing up my productivity in *bash* which obviously evolved into reading up on *readline*. Here's how to get a list of Bash keyboard shortcuts — many people don't know about most of them!

	bind -p | grep -Ev not\ bound\|self-insert\|do-lowercase-version

This is for the *Emacs editing mode* of the *readline(3)* library which is the standard input form in anything that uses *readline*.

You can also use the *Vi editing mode* by issuing the following command:

	set -o vi

This has of course got a completely different set of keyboard bindings. Should you wish so, you can change to the *Emacs editing mode* again with the following:

	set -o emacs

If you don't know *Vi* or *Vim* that well, and get "lost" in the editing mode (the keyboard doesn't type what you think it's supposed to) just press Ctrl-C to cancel the current line and enter insert mode on a fresh line buffer. 

You can also use *Alt-Ctrl-j* to switch back and forth — this works also in programs other than *bash*. If you are in *Vi mode*, you can also go to the *Emacs mode* with *Ctrl-e*.

### The GNU Readline Library
All this keyboard control is made possible thanks to the *[GNU](http://www.gnu.org/) readline* library. It is very popular; among with *ncurses(3)* it makes up almost all the interactive user interfaces in text-mode programs. Therefore, you can use these shortcuts in the Python interpreter, in *debugfs*, in *gdb*, in mail programs, and pretty much anything with a command line — it's ubiquitous. Some people [don't want to use it](http://en.wikipedia.org/wiki/GNU_readline#Choice_of_the_GPL_as_GNU_Readline.27s_license) because it's licensed under the GNU GPL, and therefore not compatible with the license of the software they are making — and they end up making [clones](http://hackage.haskell.org/package/haskeline-0.6.4.3) that [work](http://www.astro.caltech.edu/~mcs/tecla/index.html) [better](http://www.thrysoee.dk/editline/) or [worse](https://issues.sonatype.org/browse/OSSRH-1735). Of course, the best comes to those that [share](http://www.gnu.org/licenses/gpl-howto.html).

### The Vi-style bindings - Fairly Neglected
There is a *Vi mode* for readline; however, it is not very powerful. In the words of one very respected developer, *"It smacks of a sort of long-after-the-fact retrofit with little or no [modification] to allow it to actually function"*. It is heavily neglected, which shows in the fact that that its description is only about 2% of the manual *section* that it is on.

When you enter the *Vi mode*, you are immediately placed in the *insert mode*. Press *Esc* to exit to command mode; there you can naturally use *h*, *j*, *k*, and *l* to navigate the line and the history. If you press *Enter*, the line gets executed and you are placed in insert mode again.

The default list of commands in insert mode is a bit shorter than of the *Emacs mode* of *readline*; this is undoubtedly due to the fact that most people don't even know the *Vi mode* exists at all. This does not, however, include the commands possible in the *command mode* (sometimes called *movement mode* by *readline*).

	"\C-j": accept-line
	"\C-m": accept-line
	"\eOD": backward-char
	"\e[D": backward-char
	"\C-h": backward-delete-char
	"\C-?": backward-delete-char
	"\eOH": beginning-of-line
	"\e[H": beginning-of-line
	"\C-i": complete
	"\e[3~": delete-char
	"\eOF": end-of-line
	"\e[F": end-of-line
	"\eOC": forward-char
	"\e[C": forward-char
	"\C-s": forward-search-history
	"\C-n": menu-complete
	"\C-p": menu-complete-backward
	"\eOB": next-history
	"\e[B": next-history
	"\eOA": previous-history
	"\e[A": previous-history
	"\C-v": quoted-insert
	"\C-r": reverse-search-history
	"\C-t": transpose-chars
	"\C-u": unix-line-discard
	"\C-w": unix-word-rubout
	"\C-d": vi-eof-maybe
	"\e": vi-movement-mode
	"\C-y": yank

In the *command mode*, many of the basic bindings apply, and there are some additions too. Instead of listing them all, I will link you to [an excellent cheat sheet](http://www.catonmat.net/download/bash-vi-editing-mode-cheat-sheet.pdf) by [Peter Krumins](http://www.catonmat.net).

You can edit the current line in your *$EDITOR* by pressing v. By default, this puts you in *vi*; enter commands one per line, and when you exit it (e.g. with *:wq* or *ZZ*) they will be executed. This is great for editing a longer command sequence. The commands get put in the history as separate entries.

The *Vi mode* is not so terrific; the *Emacs mode* is much better.

### The Emacs-style bindings
When checking out the bindings for the Emacs mode, note that *\C* is the Control key, and *\e* is the Escape key. However, most of the time pressing Alternative and then a key sends Escape and then that key; so if there's only one *\e* mentioned you can press the Alternative key instead. There's a lot of them, there is even a [PDF cheat-sheet](http://www.catonmat.net/download/readline-emacs-editing-mode-cheat-sheet.pdf) — yet again by Peter Krumins. Not all the bindings are the same for me, though. On a default *bash* on my computer the command named above puts out the following:

	"\C-g": abort
	"\C-x\C-g": abort
	"\e\C-g": abort
	"\C-j": accept-line
	"\C-m": accept-line
	"\C-b": backward-char
	"\eOD": backward-char
	"\e[D": backward-char
	"\C-h": backward-delete-char
	"\C-?": backward-delete-char
	"\C-x\C-?": backward-kill-line
	"\e\C-h": backward-kill-word
	"\e\C-?": backward-kill-word
	"\e\e[D": backward-word
	"\e[1;5D": backward-word
	"\e[5D": backward-word
	"\eb": backward-word
	"\e<": beginning-of-history
	"\C-a": beginning-of-line
	"\eOH": beginning-of-line
	"\e[1~": beginning-of-line
	"\e[H": beginning-of-line
	"\C-xe": call-last-kbd-macro
	"\ec": capitalize-word
	"\C-]": character-search
	"\e\C-]": character-search-backward
	"\C-l": clear-screen
	"\C-i": complete
	"\e\e": complete
	"\e!": complete-command
	"\e/": complete-filename
	"\e@": complete-hostname
	"\e{": complete-into-braces
	"\e~": complete-username
	"\e$": complete-variable
	"\C-d": delete-char
	"\e[3~": delete-char
	"\e\\": delete-horizontal-space
	"\e-": digit-argument
	"\e0": digit-argument
	"\e1": digit-argument
	"\e2": digit-argument
	"\e3": digit-argument
	"\e4": digit-argument
	"\e5": digit-argument
	"\e6": digit-argument
	"\e7": digit-argument
	"\e8": digit-argument
	"\e9": digit-argument
	"\C-x\C-v": display-shell-version
	"\el": downcase-word
	"\e\C-i": dynamic-complete-history
	"\C-x\C-e": edit-and-execute-command
	"\C-x)": end-kbd-macro
	"\e>": end-of-history
	"\C-e": end-of-line
	"\eOF": end-of-line
	"\e[4~": end-of-line
	"\e[F": end-of-line
	"\C-x\C-x": exchange-point-and-mark
	"\C-f": forward-char
	"\eOC": forward-char
	"\e[C": forward-char
	"\C-s": forward-search-history
	"\e\e[C": forward-word
	"\e[1;5C": forward-word
	"\e[5C": forward-word
	"\ef": forward-word
	"\eg": glob-complete-word
	"\C-x*": glob-expand-word
	"\C-xg": glob-list-expansions
	"\e^": history-expand-line
	"\e#": insert-comment
	"\e*": insert-completions
	"\e.": insert-last-argument
	"\e_": insert-last-argument
	"\C-k": kill-line
	"\ed": kill-word
	"\C-n": next-history
	"\eOB": next-history
	"\e[B": next-history
	"\en": non-incremental-forward-search-history
	"\ep": non-incremental-reverse-search-history
	"\C-o": operate-and-get-next
	"\C-x!": possible-command-completions
	"\e=": possible-completions
	"\e?": possible-completions
	"\C-x/": possible-filename-completions
	"\C-x@": possible-hostname-completions
	"\C-x~": possible-username-completions
	"\C-x$": possible-variable-completions
	"\C-p": previous-history
	"\eOA": previous-history
	"\e[A": previous-history
	"\C-q": quoted-insert
	"\C-v": quoted-insert
	"\e[2~": quoted-insert
	"\C-x\C-r": re-read-init-file
	"\C-r": reverse-search-history
	"\e\C-r": revert-line
	"\er": revert-line
	"\C-@": set-mark
	"\e ": set-mark
	"\e\C-e": shell-expand-line
	"\C-x(": start-kbd-macro
	"\e&": tilde-expand
	"\C-t": transpose-chars
	"\et": transpose-words
	"\C-x\C-u": undo
	"\C-_": undo
	"\C-u": unix-line-discard
	"\C-w": unix-word-rubout
	"\eu": upcase-word
	"\C-y": yank
	"\e.": yank-last-arg
	"\e_": yank-last-arg
	"\e\C-y": yank-nth-arg
	"\ey": yank-pop

Some interesting bindings:

- *Ctrl-r* which enters history search mode; press it and then type something to search for previous commands; press *Ctrl-r* again to browse for the next hit.
- *Ctrl-o* which is *operate-and-get-next* — if you are in the history, you can press *Ctrl-o* in order to issue the command you are at *and get the next one*. Major time saver when redoing things by hand over and over.
- *Ctrl-a* and *Ctrl-e* which go to the beginning and end of a line.
- *Alt-f* and *Alt-b* which go forward and backward a word; you can also use *Ctrl-<Left>* and *Ctrl-<Right>*.
- *Ctrl-p* and *Ctrl-n* which go back and forward in history, just like the *<Up>* and *<Down>* arrow keys — except without lifting your hand (press the left *Ctrl* with the palm of your hand).
- *Alt-.* or *Alt-_* which repeat the last argument of the previous command.
- *Ctrl-#* comments out the current line and starts the entry of a new line.
- *Alt-/* which completes a file name and *Alt-g* which completes a word using globbing.
- *Ctrl-w* which deletes a word.
- *Ctrl-u* and *Ctrl-k* which delete the text from the cursor to the beginning and end of the line respectively.
- *Alt-=* or *Alt-?* to show possible completions of the current word. This is smart, so if it knows you're typing a file name it will only show files.
- *Alt-r* which reverts a history line if you have edited it.
- *Alt-<Space>* or *Ctrl-@* to set a mark; a sort of invisible character is placed where your cursor is. Then, if your cursor is somewhere else, press *Ctrl-x twice* and the mark will be put under your cursor and your cursor will be placed where the mark used to be.
- *Alt-Ctrl-e* to expand the line using Bash semantics; removes superfluous quotation, expands variables, etc.
- *Alt-t* to swap the word you are on and the previous one.
- *Ctrl-_* to undo the last thing you did.
- *Alt-<number> <Command>* — that is, press Alt and a digit and then continue typing a number, then execute one of the other commands — passes an argument to the command, which usually means how many times something should happen. For example: *Alt-8 Ctrl-p* goes back in history eight times; *Alt-5 x* types *xxxxx*. However, you can also start with *Alt--* (that is, Alt and minus) which will enter a negative argument; for some things, the sign of the argument is what matters, for example: *Ctrl-k* deletes everything from the cursor until the end of the line, while *Alt-- Ctrl-k* deletes everything from the cursor until the *beginning* of the line.
- *Ctrl-l* which clears the screen — that is, the line you are editing now is placed at the top of the screen, the rest is still in the scrollback.

### Resources
I have found the two cool resources for *bash* in conjunction with *readline* to be both from Peter Kumin:

- [Vi mode](http://www.catonmat.net/blog/bash-vi-editing-mode-cheat-sheet/) article,
- [Emacs mode](http://www.catonmat.net/blog/bash-emacs-editing-mode-cheat-sheet/) article.

If you would like to program completions for your own application, you can have a gander at the relevant Bash [info pages](http://real-world-systems.com/docs/bash-commandLineEditing.html#Programmable-Completion) and at the *help complete* command. I am not sure how this changes when you are using [bash-completion](http://bash-completion.alioth.debian.org/).

There is a [user manual for Haskeline](http://trac.haskell.org/haskeline/wiki/WikiDocumentation) in wiki format. You can find out more about using *[editline](http://www.thrysoee.dk/editline/)* with the command *[man editrc](http://linux.die.net/man/5/editrc)*. The [Tecla command-line editing library](http://www.astro.caltech.edu/~mcs/tecla/index.html) has fairly extensive [user documentation](http://www.astro.caltech.edu/~mcs/tecla/tecla.html).

Here is a very interesting patch which tries to [improve *readline* by displaying what editing mode it is in](https://bbs.archlinux.org/viewtopic.php?id=106428).

## ⁂

The *Vi mode* in *readline* was a fairly big disappointment. With the recent pick-up of Vi and Vim due to Vim's development as Vi was proprietary for a longer period of time, more and more people are bound to find out about it, try it, and remember never to use it.

It is my suggestion to abandon the *Vi mode* completely, set it in stone as it is, and start working on something like a *Vim mode*; undoubtedly, since the days when the *Vi mode* has been created, the Vi world has progressed immensely due to various clones of the original editor. Some improvements could be:

- first and foremost, do not make it a clone of Vi or Vim; instead, apply the *Vi philosophy* of editing to the actions that a user takes in a shell, command-line, etc. This would mean designing the different interactions from the ground up rather than "making that key do what it does in Vi".
- a visual mode without starting an editor.
- a different *.* command, which would repeat entered readline lines instead of repeating editing commands.
- better history editing; an equivalent of *Ctrl-o* from the *Emacs mode*; Vim's line completion and search.
- better *Ctrl-n* and *Ctrl-p* with the use of menus, like in Vim.
- possibly Vim's *wild mode* — a lot of people like it (though I don't use it personally).
- buffers.
- some ability to directly edit arguments to the command being edited; this would fit the *Vi philosophy*.
- a new data structure: a list, which could be like Bash arrays, and which would allow the user to run the same readline command expanded multiple times with different arguments.
- a lot of inspiration from Vim's *Ctrl-X mode*.
- inspiration from *Vimperator*, a Vim-style user interface for Mozilla Firefox.
- inspiraton from *vifm(1)*, although it too has got the burden of being built as a clone, not as a redesign in *Vi philosophy*. For example, to run a command you have to type *:!your-command* which is too much typing. There is no way to quickly cd somewhere special, although *h* and *l* navigate up and down the directory tree which is a good application of the *Vi philosophy*.
- a command-entry mode (like pressing *:* in Vim).

The basic things you do in bash should have easy commands. For example:

- a key for *cd* (*g* would be a good match)
- going backward and forward in working directory history (for example *Ctrl-I* and *Ctrl-O*)
- opening a file in the *$EDITOR* (for example :e)
- *cp* and *mv* could get love too
- a visual editor of file selection to select files as arguments to mv, cp, grep, etc using path globbing and regex searches; editing a tree as the data structure, which would then be translated to a list of globs; a possibility to store these tree structures in named buffers. In an interpreter (such as the Python interpreter), this mode could select the names of objects in the current scope for processing. Don't use *v*, *V*, and *Ctrl-v* because this is for visually selecting (and editing) lines.
- the ability to call up output of previous commands in *vim*, and to paste it into the currently edited line. This would be possible if there's a single-key command to run the current line and capture its stdout into a named buffer.
- naturally, the ability to define new keyboard commands for when you use *readline* your own application.

Above are just several ideas I have had in the first few minutes of thinking about this — I'm sure you could make a Vi-style mode of interaction with a command line as complex as Vi and Vim are; there is no doubt this could improve productivity immensely for a user of such a system.
</markdown>
