Title: Recovering damaged Linux installations with debsums
<markdown>
*Please note: the code listings in this article display completely wrong. In particular some redirects are missing. You can access the [full version of this article at my BitBucket](https://bitbucket.org/cheater/articles/src/tip/checking-system-consistency-with-debsums.md). I have emailed Posterous support about this already, hopefully they will respond to this promptly. If this does not happen I will have to consider some other way of posting my articles, and perhaps moving away from Posterous to another service altogether.*

I have recently been [recovering a broken ext3 file system](http://cheater.posterous.com/automating-e2fsck) with Ubuntu [GNU/Linux](http://stallman.org/) on it. The file system has been repaired and returned to a healthy state, which still did not mean that the files on it and in specific the operating system were in good shape or even that the computer would not erase all its hard disks and explode if I booted from that partition.

Thankfully we have packages and package managers. In Ubuntu and every other Debian based GNU/Linux distribution, every vital piece of software is installed using a deb package (this is similar in RPM based distributions such as Redhat, Fedora and SuSe). There is a tool to check the validity of files installed by a program called *[debsums(1)](http://manpages.ubuntu.com/manpages/natty/man1/debsums.1.html)*. Apparently RPM based distributions can do this with the rpm command, but I have never done this. This guide will be about debsums for deb based packages.

First off I have hooked up the hard drive to another PC, mounted the file system, and have used the *debsums* command line like this:

    debsums -sg -r /media/broken -d /media/broken/var/lib/dpkg -p /media/broken/var/cache/apt/archives 2&gt;debsums.err

This reported missing and broken files in the *debsums.err* log file. After reading through it I have noticed that there are no important packages that are broken (kernels, important libraries or binaries) and I have hooked up the hard drive to a PC and booted from it. To do that I have had to install *grub* using the Ubuntu DVD install disk's system rescue option.

Now I have had a problem: some of the package archives were missing. I need the exact same version as is installed in order to check the files. In addition to this, the system had been broken for about half a year without being used. In the Debian world this supposedly means that the packages are long upgraded, but you can still get them using *debsnap -a* somehow. I am not sure how to get the exact version of a package archive under Ubuntu, but I have thought: Ubuntu 10.04 is an older, *long-term-support* release; most packages in it are old anyways; let's see if the packages have changed at all in the last few months. To compare the installed version of a package to the available version I did the following:

	while read i; do
	  echo -n $i" ";
	  new=$(apt-cache policy $i | grep Candidate | sed -e 's/  Candidate: //');
	  installed=$(apt-cache policy $i | grep Installed | sed -e 's/  Installed: //');
	  if [ "x$new" = "x$installed" ]; then
	    echo "OK";
	    fi;
	  done &lt; B.Packages\ which\ could\ not\ be\ verified  | grep -v OK

As it turned out, *all* the packages were in their past versions! So I only had to download the files. The command *apt-get -d install* only downloads the files, it does not change the state of the *apt* database, for example whether a package was installed automatically or manually. If you want to see which of the packages were automatically installed you can run a command similar to:

	while read i; do
	  automatically=$(aptitude show $i | grep "Automatically installed: " | sed "s/Automatically installed: //");
	  echo $i $automatically;
	  done &lt; C.files\ missing\ from\ installed\ packages\ \(second\ attempt\)

After doing that I could make a "proper" run of *debsums* on that computer:

	debsums &gt; debsums2.out 2&gt;debsums2.err

The *out* file was empty and the *err* file contained both: complaints about package archives still missing, and complaints about modified files. At this point I thought the packages that were still missing were ones that I have installed by hand, but apparently some have slipped through my hands and it turned out some of the packages were indeed not available in their former versions. I decided to segregate the log file a little:

	grep "md5sums" debsums2.err &gt; debsums2.err.md5sums # missing packages
	grep -v "md5sums" debsums2.err &gt; debsums2.err.other # broken files
	grep -o "from.*package" debsums2.err.other | awk '{print $2}' | sort -u &gt; C.files\ missing\ from\ installed\ packages\ \(second\ attempt\)

I tried reinstalling one package first, using:

	apt-get --reinstall install libproj0

I got some ldconfig errors:

	ldconfig deferred processing now taking place
	/sbin/ldconfig.real: /usr/lib/libvibgif.so.6 is not an ELF file - it has the wrong magic bytes at the start.
	
	/sbin/ldconfig.real: /usr/lib/libnetentr.so.6 is not an ELF file - it has the wrong magic bytes at the start.
	
	/sbin/ldconfig.real: /usr/lib/libvibgif.so.6 is not a symbolic link
	
	/sbin/ldconfig.real: /usr/lib/libkdeinit4_kmplayer.so is not a symbolic link

As it turned out some libraries were broken — I knew of that already — and I decided that it was best to fix libraries first, and only afterwards install all the binaries and other stuff. In order find all issues with libraries I ran:

	ldconfig

I then put the reported file names in *ldconfig-errors.txt*, on one line each. Then I did:

	while read i; do dpkg -S "$i"; done &lt; ldconfig-errors.txt

The output was:

	libncbi6: /usr/lib/libvibgif.so.6
	libncbi6: /usr/lib/libnetentr.so.6
	libncbi6: /usr/lib/libvibgif.so.6
	kmplayer: /usr/lib/libkdeinit4_kmplayer.so

I reinstalled all packages mentioned by running the following twice; I did it two times to make sure nothing was linking against a broken library:

	apt-get install --reinstall libncbi6 kmplayer

Unfortunately, not all ldconfig errors could be ironed out. The *kmplayer* *so* installed by *kmplayer* cannot be fixed, but in this instance that seems to be "normal":

	objdump -x  /usr/lib/libkdeinit4_kmplayer.so

output:

	/usr/lib/libkdeinit4_kmplayer.so:     file format elf32-i386
	/usr/lib/libkdeinit4_kmplayer.so
	architecture: i386, flags 0x00000150:
	HAS_SYMS, DYNAMIC, D_PAGED
	start address 0x0000fb10
	(...)

Next, I continued reinstalling the packages. First I have reinstalled all
libraries by running the following twice:

	apt-get install --reinstall $(grep "lib" C.files\ missing\ from\ installed\ packages\ \(second\ attempt\) | tr '\n' ' ')

Then, I have just reinstalled every missing package (never mind that it reinstalls the libs again, never too many reinstalls!):
	apt-get install --reinstall $(tr '\n' ' ' &lt; C.files\ missing\ from\ installed\ packages\ \(second\ attempt\))

I got warnings such as:
	Reinstallation of java-gcj-compat-dev is not possible, it cannot be downloaded.
	Reinstallation of kfilereplace-kde4 is not possible, it cannot be downloaded.
	Reinstallation of kimagemapeditor-kde4 is not possible, it cannot be downloaded.
	Reinstallation of klinkstatus-kde4 is not possible, it cannot be downloaded.
	Reinstallation of kode is not possible, it cannot be downloaded.
	Reinstallation of kommander-kde4 is not possible, it cannot be downloaded.
	Reinstallation of kxsldbg-kde4 is not possible, it cannot be downloaded.

The packages mentioned need to be re-installed by hand.

I have also checked the items found by my *[fscheck](https://bitbucket.org/cheater/fscheck/)*. Here is a small selection:

	 [[(10301783, '/usr/share/man/man8/groupadd.8.gz'),
	  'invalid inode-&gt;i_extra_isize (47201)\r'],
	 [(10301790, '/usr/share/man/man8/grpconv.8.gz'),
	  'invalid inode-&gt;i_extra_isize (27596)\r'],
	 [(10301788, '/usr/share/man/man8/groupmod.8.gz'),
	  'invalid inode-&gt;i_extra_isize (10581)\r'],
	 [(10301787, '/usr/share/man/man8/groupdel.8.gz'),
	  'invalid inode-&gt;i_extra_isize (24484)\r'],
	 [(10301789, '/usr/share/man/man8/grpck.8.gz'),
	  'invalid inode-&gt;i_extra_isize (4287)\r'],
	 [(10301792, '/usr/share/man/man8/pwck.8.gz'),
	  'invalid inode-&gt;i_extra_isize (16045)\r'],
	 [(10301791, '/usr/share/man/man8/grpunconv.8.gz'),
	  'invalid inode-&gt;i_extra_isize (28165)\r']]

After creating a file with one filename on each line and using...

	while read i; do dpkg -S "$i"; done &lt; logfile

...I have also identified that *passwd* needs to be reinstalled — it was missing manpages.

Next, I have also analyzed the files reported by *fsck*:

	grep -E 'File.*\(inode' ../fsck | sed 's/^File //' | sed -e 's/\ [(]inode.*$//' | sort -u | less &gt; D.broken\ files\ reported\ by\ fsck

I have had to remove the one line at the top that said *...*, that was from inodes without associated paths. Then I did the usual:

	while read i; do dpkg -S $i; done &lt; D.broken\ files\ reported\ by\ fsck

I have then reinstalled all packages mentioned in the output. They included important packages such as *at*. In doing this, I have reinstalled a few packages yet again. You can improve on that if you don't want to reinstall packages over and over. For example, you could keep track of the packages you have gone over in a log file.

Additionally, I have noticed a lot of the files were in *wine, so I have had to reinstall them with *winetricks*.

I have also noticed that a few files reported by *fsck* were *.pyc* files. They are files generated by *Python* when it first interprets a file containing source code; the initial parsing process takes a small, but measurable amount of time and these cache files remove this delay on subsequent executions of a program. They can be deleted without any issue. However, I am not aware that Python does any checksumming on those files, which would mean that executing those programs could lead to program crashes.

I have deleted them using the following command:

	grep '.pyc$' D.broken\ files\ reported\ by\ fsck | while read i; do rm -vf $i; done

The next thing to do was to reinstall the last packages which could not be checked. The following command came in handy:

	debsums -gl

The first package was *google-chrome-stable*. Running *apt-cache policy google-chrome-stable* told me that the package version I have installed is not in the repository anymore; I would have to find the file myself. I have googled for *google-chrome-stable 10.0.648.205-r81283* (this is the version *apt-cache* has reported) but I could not find the package archive. This means that I could not get the file anymore. The best bet at this point is to reinstall the package with the newest version using:

	apt-get install --reinstall pkgname

Finally it is time to reboot the system and run *debsums -sg* again to see if it still reports any errors. There still were errors — reported by the same packages as before. I reinstalled those packages in the previous steps, which indicates to me that those packages will never be reported as correct.

That's all, folks! Next up, I have to figure out how to do RAID so that this sort of thing doesn't happen again.
</markdown>
