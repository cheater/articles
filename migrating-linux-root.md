<markdown>
I have recently [recovered](http://cheater.posterous.com/automating-e2fsck) a broken installation of Ubuntu [GNU](http://www.gnu.org/philosophy/who-does-that-server-really-serve.html)/[Linux](http://lwn.net/Articles/457539/) that has had [bad blocks](http://en.wikipedia.org/wiki/Rash). After rescuing the files and [repairing the OS](http://cheater.posterous.com/debsums) it was time to put the files on a fresh file system; in my opinion an ext3 that has had such major repair done to it as what fsck has done, and in general an ext3 that had to be repaired due to bad blocks, cannot be trusted with data.

### Decisions, decisions — the file system
There was the question of what file system I should use. I was told that *btrfs* is cool, and I admit that it has some really nice ideas, especially concerning the ability to check the system while it's in use. It is however only supported well in very recent kernels that Ubuntu does not have yet. I have quickly mused the idea of compiling my own 3.0.1 Linux for Ubuntu, but with kernel.org gone it's just no fun, and I don't like to mess with my OS too much anyways. Additionally, *btrfs* is still somewhat beta, so let's revisit that idea in a year or two.

The *ZFS* has cool ideas and is mature but due to its restrictive license cannot be used with the GNU operating system and therefore it's not *really* supported except for a seldom-used userspace driver that is of no use to me.

Unfortunately, *reiserFS* was largely abandoned by the open source community, which seems bent on further destroying a man whose life was destroyed by insanity. Is it really OK to condemn people who are obviously not handling their life well to the treatment of the cold shoulder? No one sits in his lounge chair sipping tea with milk at 1 PM, slightly bored, with but the sound of the clock ticking, and after reading the sports section in a gazette thinks *"Golly, I know what I should do! Let me murder somebody!"*. It's an illness like any other, and part of the recovery process is to condemn the acts that it induced in the individual. It's definitely a big misunderstanding to abandon and even condemn the person's work that is not in any way connected to their illness, especially given the extent of damage it does to that person's life. Either way this immature treatment of the project has set back the status quo of GNU/Linux file systems by at least half a decade.

Finally I got to *ext4*, which is pretty much like *ext3* but made for bigger files. Since I don't use so many big files — my FS was currently *ext3* anyways and it handled things well — I have decided to use *ext4* in a mode which makes it backward compatible with *ext3*. The *fourth extended file system* is just the *third extended file system* with *extents* and some additional stability tacked on (notably checksumming of the journal transactions); the *third extended file system* is the *second extended file system* with a few additions such as the journal. I pretty much wanted to stay with *ext3*, but the additional stability *ext4* could afford me was a boon. It is, as I understand, pretty much the same code base either way, just with some different options.

### Creating the new *ext4* partition
I have created an *ext3*-compatible *ext4* partition with this command:

	mkfs.ext4 /dev/sda4 -Ohas_journal,ext_attr,huge_file,flex_bg,uninit_bg,dir_index,filetype,extra_isize,resize_inode,^extent,^dir_nlink -L Home -m 10 -U "91998c61-9582-4840-9ef9-5c467deaf1f8"

The *UUID* (that is, the parameter to the *-U* option) needs to be the same as the source partition; it can be checked in this way:

	tune2fs /dev/source_partition -l | grep UUID

It needs to be the same because some programs (most notably *mount*) refer or can refer to partitions by their *UUID*, which would mean parts of your system would still be using the old partition. You don't want that! The author has experienced a few confusing moments when that once happened.

### Tuning it for compatibility with *ext3*
The *-O* option changes the file system options that will be used. I have turned off the option *extent* for two reasons; firstly, I want to be able to recover the file system in case of a crash, and I don't know what is involved when extents are used — I don't even know if the e2tools package is suited for serious recovery work there; and secondly, I want the FS to be mountable as ext3. I have also removed the *dir_nlink* option because I do not need the theoretical possibility of having more than 64000 entries in one directory; and I don't like the theoretical possibility that in case this happens, that directory becomes corrupt — the reference count in it stops being increased, so that if you delete the extra files again, you might end up deleting a directory which shouldn't be deleted, and possibly contains data you want to keep around.

### Copying the data
Next, I mounted both partitions on a separate file system. It's probably a much better idea to mount the source partition as read-only, but I didn't do that since I was on a LiveDVD and couldn't imagine anything wanting to write there. The partitions are *Fixed* (the old partition) and *Home* (the new, empty partition).

I moved out the *lost+found* directory on the old file system. I didn't want it to overwrite the one on the new FS, partly because that directory is given a special inode number (11) when the file system is being created:

	sudo mv Fixed/lost+found{,_}

Then, I have copied the files over. I have first tried a simple *cp -rv*, but that didn't work — all files belonged to root afterwards. After consulting *man* I decided to use the *-a* switch which is a shorthand combination of other options that is made for this situation. Additionally, I wanted sparse files not to become inflated so I used the *--sparse=auto* option:

	cp -av --sparse=auto Fixed/* Home/. > linux-copy.log 2>linux-copy.err

In order to keep track of progress I have created this quick one-liner:

	watch -n1 target=\$\(df /media/Home \| tail -n1 \| awk '{print\ \$3}' \)\; source=\$\(df /media/Fixed \| tail -n1 \| awk '{print\ \$3}' \)\; awk "BEGIN{printf\(\\\"%d%%\\\n\\\",\ 100*\$target/\$source\)}" \; df -h /media/Home /media/Fixed\; echo Errors: \$\(wc -l linux-copy.err\): \; tail -n2 linux-copy.err\; echo \; echo Latest file:\; tail -n2 linux-copy.log \| head -n1

Notice the terse elegance afforded by the escaping. It only took me about 100 tries to get all the slashes right. Interestingly, the progress indicator stopped at 99% — the file system ended up being a whopping 69536K smaller than the source. You could store about a [lifetime of music in *module format*](http://modarchive.org/index.php?request=view_chart&query=featured) in the space afforded by that.

### FSCK!
Now it's time to fsck the file system again. You need to force it to run:

	fsck /dev/sdd4 -f

Apparently, only 5.8% of my file system was non-contiguous — I can live with that, although it begs the question: how did that happen? The *cp* command should copy one file after the other — they should not be interleaved or interspersed in any way. Is there some other mechanism at play here?

### Make it boot
This step is basically swapping this partition for the old one. First you need to change the *UUID* of the old partition if you are keeping it for the time being:

	tune2fs -U random /dev/sdd1

In otder to see the *UUIDs* of all partitions you can do the following:

	fdisk -l | grep -iv swap | awk '$6=="Linux"{print $1}' | while read i; do echo  $(tune2fs -l $i | grep UUID | awk '{print $3}') $i; done | sort

It is always a good idea to keep the old partition at least for some time, if you should notice something's broken. I find one month is a good time to figure out this sort of thing. After maybe the first week of intensive use, you can keep the partition as a compressed image.

If your boot loader wasn't set to select the boot partition by *UUID*, then the next step would be to reconfigure the bootloader. I chose to simply reinstall *grub* using the Ubuntu installation disc since that is easy.

Afterwards the system should boot OK — check if everything works and have fun! :)
</markdown>
