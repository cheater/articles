Title: How to build stuff using autoconf and automake (configure.in, configure.ac, Makefile.am)
slug: autotools.md

<markdown>
Here's the deal: building stuff that uses autotools isn't just *./configure && make && make install*. I keep forgetting how to use the build system and there's no page on the net that explains it clearly.

## How to
It seems fairly easy:
	aclocal
	autoconf
	autoreconf -vi
	automake
	./configure
	make

Of course, the only source for the above invocation was threads of people failing to build software that ended up experimentally recreating the steps necessary. I may have missed something, so make sure to add info in the comments.

When done with the above list, you can do *make install* — but really, who does that nowadays? Instead use *checkinstall* to catch the files being installed into a package that can later be uninstalled cleanly, reinstalled, and so on.

#Don't forget the packages
If you're getting errors such as:
	error: possibly undefined macro: AC_PROG_LIBTOOL

then you might have to install the *libtool* package.

# ⁂
Seriously, would it hurt GNU to include info for users on how to use autotools? The *autoconf* manpage doesn't even *mention* the *aclocal* tool. Come on guys, get your heads out of your asses, start being user friendly. Having to fight your computer is so trite.
</markdown>
