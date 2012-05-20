Title: How to build stuff using autoconf and automake (configure.in, configure.ac, Makefile.am)
slug: autotools.md

<markdown>
Here's the deal: building stuff that uses autotools isn't just *./configure && make && make install*. I keep forgetting how to use the build system and there's no page on the net that explains it clearly. You've got lots of things to choose from, including *aclocal*, *autoconf*, *autoreconf*, *automake*, etc. Myself, I got rather confused.

## How to
OK, it's easier than I thought:
	autoreconf -fvi
	./configure
	make

That is, you only do *autoreconf -fvi* before the usual commands. Special thanks to Reddit user Rhomboid who pointed out you can do that instead of invoking all steps by hand.

When done with the above list, you can do *make install* — but really, who does that nowadays? Instead use *checkinstall* to catch the files being installed into a package that can later be uninstalled cleanly, reinstalled, and so on.

#Don't forget the packages
If you're getting errors such as:
	error: possibly undefined macro: AC_PROG_LIBTOOL

then you might have to install the *libtool* package.

# ⁂
Seriously, would it hurt GNU to include info for users on how to use autotools? The *autoconf* manpage doesn't even *mention* the *aclocal* tool or that you probably just want to run *autoreconf -fvi*. Come on guys, get your heads out of your asses, start being user friendly. Having to fight your computer is so trite.
</markdown>
