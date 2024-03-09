title: How to recover a lost profile file (profile.lua) after a crash in Into The Breach - data recovery 
slug: into-the-breach-data-recovery

Recently my Steam Deck ran out of battery while I had Into The Breach running, and when I started it again to play it, it told me that my profile file was lost.

The profile file has a one-stage rolling backup, similar to save games. This means normally there will be a file called something like profile.lua.backup, but sometimes that doesn't work out. To be perfectly honest that's some bad programming on part of Subset Games. I got in touch with them to see if they'd like me to walk them through how to run a multi-stage rolling backup on this file, as I'm a dev myself, so who knows, maybe they'll accept it. I was talking to a person called Isla who runs their customer service and is also a forum mod, but I haven't heard back from them yet - but it's only been a few days so far.

Anyways, here's what you want to do if Into The Breach crashes and wipes your profile file. I hope this helps someone who is really attached to their progression.

# Step 1. Don't do anything else

Don't launch any programs, don't run any operations, updates, file system checks, etc. Don't write to the disk in any way.

# Step 2. Turn off power

Don't even shut down the deck using the menu option, just hold the power long enough that the deck shuts down. Do this as quickly as possible.

# Step 3. Create a Ubuntu live USB stick

On another computer, create a live usb stick that will boot Ubuntu. It needs to be a plain USB stick at least 5 GB size, but I would suggest something that is at least 8GB just in case. Sometimes, usb sticks end up being smaller than advertised. You will also need a little space for the files you recover.

Here are instructions on how to do this on:
- [Windows](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows)
- [Linux](https://ubuntu.com/tutorials/create-a-usb-stick-on-ubuntu)
- [MacOS](https://ubuntu.com/tutorials/create-a-usb-stick-on-macos)

# Step 4. Remove your SD Card

If your SD card is in the deck, it will confuse the live usb stick bootloader and you will get a message like `grub error: you need to load the kernel first` while trying to load Ubuntu.

# Step 5. Charge up your Deck

Leave it hooked up to a charger to charge the battery up (don't turn it on yet!). When you feel like it's charged up, you can continue.

# Step 6. Plug in your USB stick

You can plug the stick in directly if it's a USB C flash drive, or you can plug it in via a USB hub or a dock. It doesn't need to be a Steam Deck dedicated dock. It's preferable to do this in docked mode because you want the Steam Deck to have power while you're doing the next steps.

If you're using a hub or a dock, try using a USB 3 or USB 3.1 port if they're available for the USB stick, as this will make Ubuntu boot faster.

# Step 7. Hook up a keyboard

It will be much easier to type all of the stuff you need to type using an actual physical keyboard, but you can also use Ubuntu's on screen keyboard for at least the first few steps. You will absolutely require a keyboard once you're editing files with Vim, but you can get pretty far to where you know if there's any recoverable data with just the on screen keyboard, even if it'll be painful.

# Step 8. Boot into the usb stick

Hold the volume down and keep holding it, and press the power button once and release the power button while still holding the volume down button. You will see the Steam Deck logo for a while, keep holding volume down until you see a text-mode boot selection screen. Using your dpad select your usb stick, and press the A button on the right side of the Steam Deck to boot it.

# Step 9. Boot into the Ubuntu installer

If everything went well, you will see another text screen where you can select what to boot - this time, the program is called grub and looks like [this](https://en.wikipedia.org/wiki/GNU_GRUB#/media/File:GRUB_screenshot.png) but with other text entries. Just press the A button on the steam deck (which amounts to Enter).

# Step 10. Select "Try Ubuntu" to run the live system.

You will be in a graphical installer with buttons that say "Try Ubuntu" on the left and "Install Ubuntu" on the right. You can find a picture of it [here](https://discourse.ubuntu.com/t/install-ubuntu-desktop-18-04/just *don't* install it!

Select "Try Ubuntu" to run Ubuntu off your USB stick, without installing it.

# Step 10 (Optional). Enable on-screen keyboard

Ubuntu recognizes the touch pads: the right touchpad is the mouse, clicking is the mouse click, the left touchpad is scrolling up and down, the right trigger is the mouse click and the left trigger is the mouse right click (context menu).

Ubuntu doesn't enable the on screen keyboard by default. Click on the little person icon in the upper right to get the accessibility menu, then click "Screen Keyboard" to enable it.

# Step 11. Open the Disks tool to see which partition your home is on.

Move the cursor to the lower left corner of the screen and click on the icon that's 9 little squares arranged in a 3x3 grid. That will bring up the Applications menu, essentially the equivalent of the Windows "Start Menu". In there, one of the icons will be called "Utilities", it has a few smaller icons in it. Inside it, open Disks.

On the left side, select your Steam Deck's internal drive. That's where your "home" file system is, which contains Into The Breach save games. Once a file gets deleted, it isn't immediately removed - instead, a new version is physically written somewhere else, and the old version remains, it just isn't accessible anymore.

In my case, I had three entries:

```
512 GB Disk
Phison ESMP...KB4C3-E13TS

512 GB Thumb Drive
Kingston DataTraveler Max

2.9 GB Loop Device
/cdrom/casp... tem.squashfs
```

Click on the one that's your internal drive.

On the right,y ou will be presented with a bunch of rectangles in line. One of them will be selected orange. They will be called "esp", "efi", "efi" again, "rootfs", "rootfs" again, "var", "var", and "home". Click "home", which is usually the largest one, usually to the right.

Below the list of partitions you will have text that goes like

```
Size: 501 GB -- XX GB free (xx% full)
Contents: Ext4 blah blah blah
Device: /dev/nvmeblahblah
UUID: 104857c-5093-0109-blah-blah
Partition Type: Linux Home Partition
```

Select the text to the right of "Device", right click it, and copy it to the clipboard. In my case, it's `/dev/nvme0n1p8` and it's likely to be the same in your case, but it could be different, so check.

# Step 12. Make the USB stick writeable

We want to recover our data, so we need a place to store it. The USB stick will have some space.

In the Applications menu, start the Terminal.

Type `sudo su` to get a root shell. Now you can do more advanced stuff but you could also lose data - so if you don't know what you're doing, take this to someone who does. Your prompt will now start with a hash character (`#`).

Remount the `/cdrom` directory so it's writeable:

`mount -o remount, rw /cdrom`

If you get an error, stop here, and figure out how to make this writeable.

# Step 13. Create a place for our work files

Still in the root shell, go to the `/cdrom` directory:

`cd /cdrom`

create a new directory and go to it:

```
mkdir recovery
cd recovery
```

We will now be working inside this directory.

It is important to write the recovered files to a drive *different* than the drive we're recovering to, or otherwise we might overwrite the data we're trying to recover right now!

# Step 14. Search for profile files on the drive

All `profile.lua` files start like this:

```
Profile = {
["version"] = 1, ["visible_name"] = "Name", ["opening"] = true, ["hide_tutorials"] = true,
["last_squad"] = 4, ["last_pilot"] = 1,
["timer"] = 101117040.000000,
```

We'll be searching for the first line.

Run the following command to search for all instances of this text:

`grep -oba 'Profile = {' /dev/nvme0n1p8 | tee -a found.log`

Make sure to replace `/dev/whatever` with what you copied out of the Disks application.

After you press enter, nothing will happen for a long while. It took 15 minutes for a single scan to happen on my 512 GB Steam Deck.

Once you have the bash prompt again, the work is over. If there was no output, either you screwed something up or all instances of profile files have been overwritten and aren't accessible anymore - sorry.

However, it's most likely that you'll have received a bunch of output that looks like this:

```
2011889664:Profile = {
2012844032:Profile = {
7459569664:Profile = {
7465869312:Profile = {
7466041344:Profile = {
7466995712:Profile = {
10387206132:Profile = {
19323088896:Profile = {
19323125760:Profile = {
92432952938:Profile = {
92444102863:Profile = {
92444105427:Profile = {
92448541384:Profile = {
92468106700:Profile = {
134454104064:Profile = {
134472327168:Profile = {
166502039552:Profile = {
166520430592:Profile = {
166520463360:Profile = {
166520582144:Profile = {
166532780032:Profile = {
166532894720:Profile = {
198663999488:Profile = {
225629884416:Profile = {
274152931328:Profile = {
274153029632:Profile = {
274153172992:Profile = {
352010555392:Profile = {
375396569088:Profile = {
375396667392:Profile = {
375396704256:Profile = {
433314639872:Profile = {
447965614080:Profile = {
450320355328:Profile = {
450320424960:Profile = {
450378604544:Profile = {
450379038720:Profile = {
450379132928:Profile = {
450379223040:Profile = {
450379288576:Profile = {
```

This is great! That's a lot of potential files to recover.

# Step 15. Copy out potential `profile.lua` files to your directory.

Run the following command to copy out a bunch of files:

`while read i; do skip="$(python3 -c "print($i//1024)")"; dd if=/dev/nvme0n1p8 of="$i".found bs=1K count=100 skip="$skip"; done < <(cat ../found.log | grep 'Profile = {' | cut -f1 -d:)`

Note the value right of `count`. I currently set it to 100, which means it will copy 100 kilobytes (102400 bytes) of data after the position at which the string "Profile = {" was found. This is because the amount of data is count * bs, and bs is set to 1K, which means 100 * one kilobyte = 100 kilobytes.

Someone sent me their profile.lua file with almost everything unlocked, but not quite everything (I think they were missing one mech squad). Their file as almost 102400 KB, just under that. So if your profile.lua was very large, you might want to increase this number, for example to 200, so that you copy out 200 kilobytes.

You will get output that looks like this:

```
102+0 records in
102+0 records out
104448 bytes (104 kB, 102 KiB) copied, 0.0409466 s, 2.6 MB/s
102+0 records in
102+0 records out
104448 bytes (104 kB, 102 KiB) copied, 0.00170147 s, 61.4 MB/s
102+0 records in
102+0 records out
104448 bytes (104 kB, 102 KiB) copied, 0.00196664 s, 53.1 MB/s
102+0 records in
102+0 records out
104448 bytes (104 kB, 102 KiB) copied, 0.001776 s, 58.8 MB/s
102+0 records in
102+0 records out
104448 bytes (104 kB, 102 KiB) copied, 0.00136369 s, 76.6 MB/s
```

etc. (I actually had my count set to 102 here).

Once that's done, running the `ls` command will find a bunch of files that might look like this:

```
# ls
10387206132.found   166520463360.found  19323125760.found   274152931328.found  375396667392.found  450320424960.found  450379288576.found  92432952938.found
134454104064.found  166520582144.found  198663999488.found  274153029632.found  375396704256.found  450378604544.found  7459569664.found    92444102863.found
134472327168.found  166532780032.found  2011889664.found    274153172992.found  433314639872.found  450379038720.found  7465869312.found    92444105427.found
166502039552.found  166532894720.found  2012844032.found    352010555392.found  447965614080.found  450379132928.found  7466041344.found    92448541384.found
166520430592.found  19323088896.found   225629884416.found  375396569088.found  450320355328.found  450379223040.found  7466995712.found    92468106700.found
```

# Step 16. Create a directory to shortlist recovered files

Run `mkdir shortlist`

# Step 17. Install Vim

We'll be using Vim. We're using it because these files will contain binary garbage at the end and Vim will display it without errors and let you delete it.

You can install Vim using this command:

`apt install vim`

At this point you'll want to read a vim tutorial and make sure you know how to edit a file, get into and out of insert mode, how to move the cursor using hjkl, how to delete characters, and how to run commands that start with a colon (`:`) like `:saveas`.

# Step 18. Open all the files in Vim as tabs:

`vim -p *.found`

# Step 19. Search around files for ones that look good

You can cycle through the tabs with the gt command (type g, then t) to go right and gT (type g, then shit and t) to go left.

Find files that look like they have all the data you want. You can move around in the file using h, j, k, l to move the cursor around. You can also use the page up and page down keys on your keyboard.

If you ever get into a weird mode because you pressed the wrong key, you want to type the Esc key a few (say, 5) times, and then press enter.

Don't worry about garbage characters at the end of the file. Just make sure the file isn't truncated.

For example, this file is truncated:

```
["difficulty"] = 0, ["victory"] = false, ["squad"] = 4, ^@^@["stat_tracker"] = {["games"] = 1, ["travelers"] = 0, ["kills"] = 0, ["islands"] = 0, ["pods"] = 0, ["total"] = 0, ["victories"] = {0, 0, 0, },^@^@^@["trackers"] = {["Global_Pilot_Three"] = 1, ["Global_Pilot_Final"] = 6, ["Global_Pilot_Unlocked"] = 1, },^@^@["achievements"] = {["Global_Pilot_Final"] = 0, },^@^@^@["last_end"] = 2, },^@["pilot"] = {["id"] = "Pilot_Aquatic", ["name"] = "Archimedes", ["name_id"] = "Pilot_Aquatic_Name", ["renamed"] = false, ["skill1"] = 1, ["skill2"] = 0, ["exp"] = 50, ["level"] = 2, ["travel"] = 15, ["final"] = 6, ["starting"] = true, ["power"] = {0, },^@["pilots"] = {"Pilot_Original", },^@^@["squads"] = {true, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, },^@^@["tutorials"] = {},^@^@["unlocked_island"] = 0, ^@["timer"] = 115290.828125, ^@["last_squad"] = 0, ["last_pilot"] = 1, ^@["version"] = 1, ["visible_name"] = "Name", ["opening"] = true, ["hide_tutorials"] = true, ^@Profile = {^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@
```

whereas this one isn't (I'm only showing the first screen ful of it)

```
Profile = {
["version"] = 1, ["visible_name"] = "Name", ["opening"] = true, ["hide_tutorials"] = true,
["last_squad"] = 4, ["last_pilot"] = 1,
["timer"] = 0.000000,
["unlocked_island"] = 0,

["tutorials"] = {},

["squads"] = {true, false, false, false, false, false, false, false, false, false, false, false, false, false, false, false, },

["pilots"] = {"Pilot_Original", },


["achievements"] = {},

["trackers"] = {["Global_Pilot_Unlocked"] = 1, },


["stat_tracker"] = {["games"] = 0, ["travelers"] = 0, ["kills"] = 0, ["islands"] = 0, ["pods"] = 0, ["total"] = 0, ["victories"] = {0, 0, 0, },


["squad0"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad1"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad2"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad3"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad4"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad5"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad6"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad7"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad8"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad9"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },
```

You can check what's under the `visible_name` key, it should be what you entered when you created the profile.

The file is considered "complete" if it ends like it should. Here's an example of what this file ended up like:

```
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad14"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["squad15"] = {["victories"] = {0, 0, 0, },
["games"] = 0, ["score"] = 0, ["kills"] = 0, },

["pilots"] = {
},

["enemies"] = {
},
},
}
^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@0\r§^[mûü^E^@^@^@w^@^@^@^P$^]<9a>^@^@^@^@1/0/https://cdn.steamstatic.com/steamcommunity/public/images/items/1263950/f5a7dffe45c8dc1578053f3ba4723f78729bd4bf.png<89>PNG^M
^Z
^@^@^@^MIHDR^@^@^@à^@^@^@à^H^F^@^@^@^Z-já^@^@^\7iTXtXML:com.adobe.xmp^@^@^@^@^@<?xpacket begin="ï»¿" id="W5M0MpCehiHzreSzNTczkc9d"?>
<x:xmpmeta xmlns:x="adobe:ns:meta/" x:xmptk="Adobe XMP Core 6.0-c002 79.164360, 2020/02/13-01:07:22        ">
 <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
  <rdf:Description rdf:about=""
    xmlns:xmpMM="http://ns.adobe.com/xap/1.0/mm/"
    xmlns:stEvt="http://ns.adobe.com/xap/1.0/sType/ResourceEvent#"
    xmlns:stRef="http://ns.adobe.com/xap/1.0/sType/ResourceRef#"
    xmlns:dc="http://purl.org/dc/elements/1.1/"
    xmlns:xmp="http://ns.adobe.com/xap/1.0/"
    xmlns:xmpDM="http://ns.adobe.com/xmp/1.0/DynamicMedia/"
    xmlns:stDim="http://ns.adobe.com/xap/1.0/sType/Dimensions#"
   xmpMM:InstanceID="xmp.iid:02ffe289-d75d-7d4f-9336-33136d8e7fab"
```

You can see there were some `squad` entries, then `pilots`, then `enemies`, and finally it ended with a lone right brace `}` on a line of its own. After that there was a bunch of binary garbage (`^@` is a null character, meaning that the drive physically has 0's in that byte). Then, another completely unrelated file started. That part doesn't matter, it's fine, we'll delete it later.

Always check the garbage at the end of files. Sometimes a complete other profile file can be hiding in there! You'll have to save that out to another file.

# Step 20. Copy out the files to the shortlist directory

When you find a file that looks promising, store it out, using a command like this:

`:saveas shortlist/1.lua`

obviously increase the number every time you do this.

Once you're done, quit vim using `:q`.

# Step 21. Compare shortlisted files

Go to the shortlist directory and compare the shortlisted files:

```
cd shortlist
vim -p *.lua
```

You will notice that each file has a `timer` key. I believe that counts the time you've been playing for. Find the one with the largest timer. Check again if it's complete. Read the file through from beginning to end and see if it follows correct LUA syntax - you need to know something about programming to understand what's going on.

One thing you can do is you can go to the very first line that is `Profile = {` and move your cursor so it's over the left brace (`{`). Then, on your keyboard, press the `%` key (usually shift + 5 on US keyboards). That should take you to the matching brace at the end of the file. If it does that, you know the file you recovered has OK syntax.

If everything is good, save it as `last.lua`:

`:saveas last.lua`

Quit Vim using `:quit`.

# Step 22. Clean up the file

Open `last.lua` with Vim:

`vim last.lua`

Using the hjkl keys or using the `%` key like described in the previous step, navigate to the final closing brace of the lua file, so that the cursor is right on top of it. If there are no more charcters after that, great, you're done! But there will likely be characters there, like in the examples above.

Press the space bar once to move one character to the right. You are now at the very beginning of the garbage data.

Press dG to delete everything until the end of the file. The garbage should now have disappeared. The status bar at the bottom left corner of Vim will tell you something like this:

`387 fewer lines`

(the number will probably be different)

You can now save the file using the `:save` command.

# Step 23. Copy the recovered file into your home directory

Shut down Ubuntu using the power menu in the top right corner.

Reinstall your SD card if you pulled it out before.

Boot into SteamOS desktop.

Using the file manager, copy the existing file from the USB stick into your home directory, so that it is under the path `~/last.lua`.

# Step 24. Restore the saved file

In a terminal, go to the directory that holds your profiles:

`cd /home/deck/.local/share/IntoTheBreach/`

List the files using `ls`. It will show you your profile dir.

go into it:

`cd profile_Name`

Move the existing `profile.lua` to something else, eg:

`mv profile.lua profile.lua.bad`

Move the lua file into place:

`mv -iv ~/found.lua profile.lua`

You should be done recovering your profile!


A similar process can be used if you lose your Into The Breach save game, but you'll have to search for a different string when using the grep command on the disk drive.

# Issues

I haven't found a way to restore the current pilot, but I'll keep on digging to see if there's any way to do that.
