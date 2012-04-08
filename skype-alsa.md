Title: How to fix Skype hogging audio hardware
Slug: skype-alsa

<markdown>
## TL; DR

Quick fix: run

	aoss skype

instead of

	skype

and you're fine.

## Skype prefers crappy PulseAudio

OK guys, whether PulseAudio is good or not, the way it works under Ubuntu means it's a crappy choice. PA Might work perfectly well under Arch or Debian — but my problems were under GNU/Linux Ubuntu 10.04.

## The problem with PulseAudio

Currently, I have several audio subsystems here. Having Alsa and PulseAudio at once isn't that uncommon. However, on my computer if I use PulseAudio then no Alsa application can access the sound outputs or inputs. I haven't bothered fixing it because with most apps I can fix it by changing the output being used; Skype defaults to PulseAudio if it's present, ignoring Alsa. Skype was the last holdout, and it was annoying me. All the guides I could find mentioned uninstalling PulseAudio or heavyweight configuration changes to Alsa, but [one of them](https://wiki.archlinux.org/index.php/Skype#B._Making_ALSA_.2B_dMix_work_for_Skype) defined a configuration which had PulseAudio *and* Alsa at the same time; they launched Skype with *aoss skype* and it worked. The command line included some other crud, but it tipped me off on *aoss*.

Apparently Alsa supplies the tool *aoss* which wraps any binary and makes it use Alsa. So, after trying: success!

## Don't use the command line every time
You can edit the Ubuntu menu entry by right-clicking on *Applications* in the Gnome Panel, and choosing *Edit menus*. There, under *Internet*, you can click on the *Skype* entry (make sure not to uncheck it) and then click the *Properties* button to the right. This should open a window called *Launcher Properties* in which you can change the *Command* field to *aoss skype*. Now you can launch Skype through the menu, and it works.

# ⁂

I'm not too big on installing corporate malware, but Skype is one of the few things you can't live without nowadays if you interface with normal humans.

There *should* be a [free](http://www.gnu.org/) fully-[p2p](http://en.wikipedia.org/wiki/Magnet_URI_scheme) solution to this problem that *just works*, but there isn't anything prominent yet. I'm not so certain it'd beat Skype unless it had very serious corporation backing.

</markdown>
