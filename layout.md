TITLE: My new keyboard layout that is similar to QWERTY

I have created [a new keyboard layout](https://bitbucket.org/cheater/us_split/overview) which can be used in GNOME with xkb. It contains an installer and a bit of instructions that should get you going fairly quickly. Installing should not require restarting X. Below are the contents of the README file for the layout, including an ASCII graphic representation of the layout.

# ⁂
"I've done a fair share of stupid things in my life, a couple of which should have put me in the grave. But here I am, typing away as if I had a brain."
-Craig Wilson

### NAME
*us_split*
an xkb keyboard layout

### ABOUT THIS PROGRAM
A keyboard layout for xkb. Geared specifically towards reducing RSI, this layout is nearly identical to QWERTY, except there is a two-key split between the *T* and *Y* columns into which symbols from the furthest right of each row are placed. This way, the hands can assume a more natural position (sort of like with Dvorak) and hitting enter is not a contortionist exercise any more.

The problem with QWERTY is, among others, that the right hand needs to move a lot — especially in the wrist — which can lead to Repetitive Strain Injury. If you spend a lot of time typing, you should be aware of this and should know that you might be hurting yourself with an inappropriate keyboard layout. This layout makes it easy for people to try out something new, while alleviating some important issues.


The changes can be seen easily at the diagrams below:

Normal QWERTY:

	` 1 2 3 4 5 6 7 8 9 0 - = Back
	Tab q w e r t y u i o p [ ] RRR
	Caps a s d f g h j k l ; ' \ RR
	Shf . z x c v b n m , . / Shift
	Ctr W Alt __________ Al RC Ctrl


This layout:

	` 1 2 3 4 5 - = 6 7 8 9 0 BBBB
	Tab q w e r t [ ] y u i o p RRR
	Caps a s d f g \ ' h j k l ; RR
	SSS B z x c v , . b n m / SSSSS
	CCC W Alt __________ Al RC CCC

The key between the left *Shift* and *z* is the *105th key*, and is a second *Backspace*. It exists on many European keyboards and some people love it, while some people hate it with their whole essence. The author has been a member of both groups.

### INSTALLATION
In order to install the layout under Ubuntu 10.04, use the included *install.py* script which will take care of the rest. This should work in newer versions as well, however has not been tested with the *Unity* desktop or *KDE* and might not work in them. This version has only been tested with *GNOME* under Ubuntu 10.04. Installation should not require restarting the X server.

In order to install on other systems, you might need to change the options of the installer. See *./install.py --help* for a description of the options. Specifically, the *xkb* directory might have moved. You are searching for a directory called *'xkb'* with subdirectories including *'rules'* and *'symbols'*, as well as a few others. You might also need to change the name of the xml file, normally *rules/evdev.xml* is used on Ubuntu 10.04 but it might be *rules/xfree86.xml* or *rules/base.xml* or *rules/xorg.xml*.

On some systems *.lst* files with the same base names are used instead. Those files are not supported by the installer, however editing them by hand should be fairly easy: in the *!layout* section you would add the line:

	us_split  USA Split

Then, you should put the *us_split* file in the *symbols/* subdirectory. Finally, a restart of your *X* server might be required. This has, however, not been tested.

### USAGE
In order to use the layout, under GNOME you would use the program *gnome-keyboard-properties(1)* which can normally be accessed in the *System* menu, *Preferences* submenu. Alternatively, it can be run from the terminal. Then, in the *Layouts* tab, you would click the *Add* button and navigate to the *By Language* tab. In this tab, select *English* as the *language* and then find *USA Split* in the *Variants* dropdown. Finally, click *Add*. You can then select the layout from the list.

### STABILITY
Note that the layout may change (even drastically!) in the future. Older versions should be available from the usual sources.

### LICENSE
This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program.  If not, see [http://www.gnu.org/licenses/](http://www.gnu.org/licenses/).
