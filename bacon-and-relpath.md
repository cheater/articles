Title: bacon, libbacon_ps1 and relpath released to improve your bash and zsh experience
Slug: bacon-and-relpath

<markdown>
I have just released my latest programs: *[bacon](http://bitbucket.org/cheater/bacon)* and *[relpath](http://bitbucket.org/cheater/relpath)*. They come to solve a frustration that I have had over this year when I had to work in deeply nested dirs for a longer period of time; they have since evolved to be much more powerful.

## bacon
> The bacon library and supporting binaries are made to help you find yourself in your file system. The idea is simple: create prefixes just like the ~ used for the home directory; use them to cd to directories easily; show the path in the bash prompt relative to those stored paths. 

> Each such stored path is called a bacon, and is represented in the file system by a symbolik link in ~/.bacons pointing towards your path of choice. You may create new bacons using bacon -n BACON, as well as remove existing bacons using bacon -e BACON. You can also list currently existing bacons using bacon -l, bacon -ll, or bacons. The last two commands are exactly equivalent.

This is very similar to the way bash treats the ~ shorthand for the home dir. So, for example, if I have a bacon called % pointing to $HOME/projects/current, and I am in the directory for the bacon project, then my PS1 will display:

	cheater@orion:%/bacon$

This is a great improvement over, say...

	cheatr@orion:~/projects/current/bacon$

This can definitely bring a big improvement over something like

	cheater@orion:/media/c22a0ea9-8b2f-4af2-a699-e79c6bbc3938/temp_working_folder_while_ubuntu_is_broken/projects/recovery

which was my prompt for half a year and which again could be something like:

	cheater@orion:=/recovery$

Naturally, longer names are allowed too, as well as names with spaces.

Additionally it allows me to go to my projects folder with the command:

	cd %

Tab completion is supported.

The thing is fairly easy to use and very native to bash, therefore integrates well.

## relpath
The bacon package requires *relpath*, another program I have just released, which is a precursor to *bacon*. You can use *relpath* to find out the relative path from one file to another.

## The manuals
Below I have placed the manuals which should come in handy. I hope you like the tools! If you end up using them, please let me know — I'd be glad to hear from anyone who likes them!

## The bacon readme
### NAME
*bacon*, *bacons*, *libbacon_ps1*

### ABOUT THIS PROGRAM
This program enables you to create prefixes for directories and refer to them.

### SYNOPSIS
	bacon [OPTION]... [-d BACON]... [-e BACON]... [-n BACON]...

	bacons

### INSTALLATION
This program requires the presence of the programs *relpath*, *realpath*, *readlink*,
*basename*, as well as *bash*.

In order to install this program you first need to install *relpath*, which can
be found at [<http://bitbucket.org/cheater/relpath>](http://bitbucket.org/cheater/relpath). Next, execute the installer
*install-bacon.sh*, which will install the executables and the library by making
symlinks to files under this directory, as well as append your *.bashrc* script.

Alternatively, just copy the files into the right places and augument your
user profile's *.bashrc* file.

### USAGE
The *bacon* library and supporting binaries are made to help you find yourself in
your file system. The idea is simple: create prefixes just like the *~* used for
the home directory; use them to *cd* to directories easily; show the path in the
*bash* prompt relative to those stored paths.

Each such stored path is called a *bacon*, and is represented in the file system
by a symbolik link in *~/.bacons* pointing towards your path of choice. You may
create new *bacons* using *bacon -n BACON*, as well as remove existing *bacons* using
*bacon -e BACON*. You can also list currently existing *bacons* using *bacon -l*,
*bacon -ll*, or *bacons*. The last two commands are exactly equivalent.

	-n BACON    create new bacon named BACON, pointing to working directory
	-e BACON    eat BACON - remove it, but only if it's a symlink or empty dir
	-d BACON    delete BACON and any contents if it was a directory
	-l[l]       list bacons; use twice for verbose listing
	-f          fry bacon
	-h          print this help

In addition to this, by putting *~/.bacons* in your *$CDPATH*, you can easily *cd* to
the chosen target directory using *bacons*. The default installation configures
*$CDPATH* in this way.

It is notable that *bacons* can be given one-character names such as *=*, *%*, and so
on, and can also have names that contain spaces. Additionally, names containing
Unicode, such as *≈* and *≋*, are interesting for identifying paths (however not so
good for being used with *$CDPATH* and *cd*).

### STABILITY
This program should be fairly stable.

### FILES
	~/.bacons       the bacons directory
	~/.bacons/*     the individual bacons

### NAME
The author first wanted to call this program *beacon*, however *beacon* was taken
already and *bacon* is close enough. Additionally, this program expands on the
idea of *librelpath_ps1*, which allows a single prefix designated by use of the
character *=*. The *=* character looks like a strip of bacon.

### HISTORY
Research indicates that *~* is prehistoric dinosaur bacon. However, no one has
eaten it and therefore the actual nature of the meat should be regarded as an
object of pure speculation.

### GOSSIP
It is said that the author has not eaten anything for two weeks while writing
the complete code and documentation for *bacon* and *libbacon_ps1*. Please invite
the author to dinner, lest he perish.

### LICENSE
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.

## The relpath readme
### NAME
*relpath*

### ABOUT THIS PROGRAM
This program puts out the relative path of two files.

### SYNOPSIS
	relpath [PATH [PATH]]

### INSTALLATION
This program requires the programs *dirname*, *basename*, *realpath*, and *bash*.

In order to install this program just run *install-relpath.sh*, which will
install the *relpath* executable and the library by symlinking to the files under
this directory. Alternatively, just copy the files into the right places.

If you would like to use *relpath* for your prompt, run *install-prompt.sh* which
will also install *relpath* and *librelpath_ps1*. Again, alternatively just open
the installer and do what it would do, manually.

### USAGE
The *relpath* program can be called with two arguments that are paths to files or
directories. The output is the relative path from the first argument to the
second argument. Additionally, if the variable *$RELPATH* is set, *relpath* can be
called with one argument or no arguments, In this case, the value of the
*$RELPATH* variable is always taken as the *source*, and the value of the only
command line argument is taken as the *destination*. If the argument is missing,
the current working directory is used.

If the relative path would go through the file system root, the absolute path
is returned instead.

The *librelpath_ps1* library can be used for your prompt (*$PS1* in *bash*). To do
this you need to use the script from the installer, which sources the library
*librelpath_ps1* and calls *$(ps1path "\w")* instead of *\w*. This then displays a
relative path just like in the home directory; except that it is relative to
the value of the environment variable *$RELPATH*. The prompt then uses *=* instead
of *~* as the prefix.

### ENVIRONMENT
The variable *$RELPATH* can allow the program to be run with one and zero
parameters, which is described in the earlier sections of this document.

### STABILITY
This program should be fairly stable. There can be speed improvements if it is
rewritten in *C*.

### LICENSE
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
</markdown>
