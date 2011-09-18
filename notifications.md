Title: Receiving notifications from the terminal

<markdown>
If you do a lot of work in the terminal, you inevitably end up doing things that take so much time you just don't want to wait. You want to be notified by the program when it's done, instead of having to check every five minutes. This applies to getting notifications when you are copying, compiling, or building things, when a batch job is done, and when you get a message in an irc client (e.g. irssi). Unfortunately this doesn't work as it should in Ubuntu GNU/Linux.

### Saved by the BEL
If you like to be driven insane you have sound notifications on. In this case, you can do the following:

	my-long-command --build; echo ^G

where the *^G* is entered in a special way: you press *Ctrl-V* to enter escaped entry mode and then press *Ctrl-G*. Works in Ubuntu in gnome-terminal.

If, however, you like your peace and quiet (or find that the default bell sound does not harmonize with your favourite esoteric jazz LP) then you might want to rely on some visual form of bells.

There are [patches](https://bugzilla.gnome.org/show_bug.cgi?id=557593) and a [gnome-terminal PPA]https://launchpad.net/~morethan/+archive/ppa() to do this. The patches have been there since 2008; might as well be 1908, they still aren't applied. Use the PPA if you want this feature, for example for your *irssi* running inside a *screen*. The author of the patch has also [written](http://mikelward.com/news/2008/11/terminal-that-tells-you-when-its-done.html) that you can send out a bell every time you get a new prompt in this way:

	PS1="\\007$PS1"

This would create no annoying bells when you are using the terminal normally, either.

### Bubbles
Ubuntu has a cool functionality for showing notification bubbles, which doesn't require hacking *gnome-terminal*. You can send them out from scripts using *notify-send(1)*. For example, this is what a compilation could look like:

	make && make-install && notify-send "Big Project" "Compilation done." || notify-send "Big Project" "Error"

The first string is the *summary* (the heading) and the second, optional one is the *body* of the text.

You might need to install *libnotify-bin* in order to have this command.

### Popups
Bubbles only work if you are there all the time and are sure you'll see them. If you're gone for even a couple seconds you might be asking yourself if you've missed it. You can have a popul which requires user interaction. The first thing to try is *zenity(1)*:

	zenity --error --text 'Done!'

	zenity --info --text 'Done!'

	zenity --warning --text 'Done!'

You can also use *zenity* for control flow, as illustrated in the *manpage*:

	zenity   --question  --title  "Alert"   --text "Microsoft Windows has been found!
	Would you like to remove it?"

If you don't have access to *zenity* you might also try *xmessage(1)* -- it's like *zenity*'s [trilobite](http://en.wikipedia.org/wiki/Trilobite) cousin; the thing is ugly but it's there. Useless for me though, since it does not follow font size settings I have made in GNOME and therefore its text is just too tiny:

	xmessage you can\'t read me, sucker

Even the window is so tiny I am afraid that I might just miss it.

Either way, you might find yourself with the popup out of focus and the terminal in front of you, waiting indefinitely for the command to yield control. Therefore it's usually a good idea to also echo something, such as:

	compile-stuff && echo done. && zenity --info --text "Done!"

This should save you a lot of grief. Have fun!

## ⁂

Unfortunately *gnome-terminal* doesn't really support bells that well. You would expect the following to hold true:

1. If the window and terminal tab sending the *BEL* are in focus, the terminal background gets flashed.
1. If the window is in focus but the tab is not, the tab title is made bold and perhaps flashes once, or keeps on flashing -- set it in preferences. Then, if you switch away from the window, the taskbar entry should become bold.
1. If the window is not in focus, the tab name is made bold and the window entry on the taskbar is also made bold.

Granted, all of those should be subject to user preferences -- there are people who just can't stand this -- but the above are not even implemented *badly*.

They are not implemented *at all*. This does not work with *gnome-terminal*: a cursory check shows that it works with various *rxvt* versions; search the *manpage* for *urgentOnBell*. It also works in *xterm(1)*, using either *xterm -pob* (mnemonic: pop on bell) or configuring it -- search the *manpage* for *popOnBell* and *bellIsUrgent*.

Terminals have always had the functionality to receive bells. Apparently the functionality exists since at least 1870 -- which makes it even funnier that GNOME still doesn't do that properly. Yet again *gnome-terminal* proves to be somewhat of a toy compared to other, "more serious" terminals like *xterm* or *rxvt*. Now if configuring them didn't suck so badly..

</markdown>
