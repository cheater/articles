Title: Debugging Firefox 7 with gdb, fixing the bug, and submitting a patch
slug: hacking-firefox

<markdown>
I have recently moved from a temporarily installed Ubuntu 10.10 back to my Ubuntu 10.04 desktop. There I have found a nasty surprise — Firefox would hang up every time I created a new tab. I have searched around, tried disabling different addons and getting the latest versions, however Firefox would still keep stumbling.

### Whip out gdb
It was time to resort to debugging the binary itself. I have first tried running Firefox in gdb. Apparently [what you need to do](http://kb.mozillazine.org/Getting_a_stacktrace_with_gdb) is:

1. Install the right debugging symbols. In Ubuntu GNU/Linux it's as easy as *aptitude install firefox-dbg*, this also works if you have Firefox Beta or Aurora installed through the PPAs. Watch out, this weighs at over 150 MB to download and nearly 400 MB on disk.
1. Run Firefox in debugging mode:
	firefox -g -d gdb
1. Attach *gdb* with:
	gdb --pid=$(pidof firefox) # use firefox-bin instead of firefox for FF6 and lower
1. When *gdb* is loaded, continue execution of Firefox using the *continue* command.
1. Cause the fault to happen. In my case it was a fairly beautiful segmentation fault. At that point the *firefox* binary will be stopped and *gdb* will show a command prompt again. There you can enter the command *backtrace* to get the backtrace and see what went wrong. Without symbols you only get the address of the instruction pointer; with symbols you get the file name of the source and the function name. Very handy.
1. Once you're done with the backtrace issue *continue* into *gdb* in order to let Firefox execute its exception handler and quit normally. Then quit *gdb* with *quit*.

### Getting the source
You can [get the source](https://developer.mozilla.org/en/Download_Mozilla_Source_Code) at the Mozilla FTP:
	ftp://ftp.mozilla.org/pub/mozilla.org/firefox/releases/

Of course, you need the exact version of the source, but you know that.

### Compiling it
The next step is to compile the source and reproduce the bug again. If it is not reproducible just keep the binary :D

First you need to run *./configure*. It might complain about some missing packages; usually it's as easy as *aptitude get libwhatever-dev* to get the headers. One exception is *yasm*: Firefox needs it to compile WebM support and add "better" support for JPEG decoding. However, GNU/Linux (Ubuntu 10.04) does not have the right version of *yasm*: the latest is in the 0.8 series, Firefox needs 1.1. You first need to *aptitude remove yasm* if you have an older version. If you have Firefox from the Mozilla PPAs then you can do *aptitude install yasm-1*. This installs */usr/bin/yasm-1*, however *configure* is looking for *yasm*. So just *ln -s /usr/bin/yasm-1 /usr/bin/yasm* and you're good to go.

When executing Firefox from the shell, I have noticed it complaining about *libxul.so* not containing some sections it needs. It was finding an older *libxul.so*; This was fixed by linking to the *libxul.so* that is found at */usr/local/lib/firefox-7.0/libxul.so*.
(describe compiling, and how to install yasm-1 as yasm)

### The Culprit
As it turns out, the bug was here:

	nsHTMLTextFieldAccessible::GetNameInternal(nsAString& aName)
	{
	  nsresult rv = nsAccessible::GetNameInternal(aName);
	  NS_ENSURE_SUCCESS(rv, rv);

	  if (!aName.IsEmpty())
	    return NS_OK;

	  if (mContent->GetBindingParent())
	  {
	    // XXX: bug 459640
	    // There's a binding parent.
	    // This means we're part of another control, so use parent accessible for name.
	    // This ensures that a textbox inside of a XUL widget gets
	    // an accessible name.
	    nsAccessible* parent = GetParent();
	    parent->GetName(aName);
	  }

	  if (!aName.IsEmpty())
	    return NS_OK;

	  // text inputs and textareas might have useful placeholder text
	  mContent->GetAttr(kNameSpaceID_None, nsAccessibilityAtoms::placeholder, aName);

	  return NS_OK;
	}

See that line where we call *parent->GetName* — the same one that *gdb* reported as the site of the segfault? That's right. There's a zero pointer there. This is fairly easy to fix:

	    nsAccessible* parent = GetParent();
	    if(parent) {
	      parent->GetName(aName);
	    }

After a recompilation the bug wasn't there anymore and we're all happy! :-)

### Giving back
Now it's time to return the favor for people who made Firefox what it is. To submit a patch you first need to check if it still exists in the repository; then register on the Mozilla Developer Network and file a bug; then attach a patch. [No patches are reviewed without associated bugs](https://developer.mozilla.org/en/Hacking_Mozilla).

The patch needs to be against the latest source (watch out — another hefty download):
	hg clone http://hg.mozilla.org/mozilla-central/ tip

Check if the bug still persists, and if yes, patch it. Then generate a patch.

In my case, the bug had been patched already. This is what the tip looked like:

	    nsAccessible* parent = Parent();
	    if (parent)
	      parent->GetName(aName);
	  }

No need to create a patch. Suppose though this were not the case and I wanted to submit a patch, it would go like this:

First, edit the file and add your changes, check the bug is fixed. File a bug to get a bug number. Go to the main directory of the source and issue *hg diff > ../diff-bugnumber.diff*. You should get something similar to this:

	diff -r 8540ca31ca8f accessible/src/html/nsHTMLFormControlAccessible.cpp
	--- a/accessible/src/html/nsHTMLFormControlAccessible.cpp	Sun Sep 18 11:22:18 2011 +0200
	+++ b/accessible/src/html/nsHTMLFormControlAccessible.cpp	Sun Sep 18 15:22:35 2011 +0200
	@@ -410,8 +410,8 @@
	     // This ensures that a textbox inside of a XUL widget gets
	     // an accessible name.
	     nsAccessible* parent = Parent();
	-    if (parent)
	-      parent->GetName(aName);
	+    if (parent) # patch test
	+      parent->GetName(aName); # patch test
	   }
	 
	   if (!aName.IsEmpty())

You can then attach the diff to the bug in bugzilla. I believe the bug can also be tagged as having a diff that needs review.

### More fun
If you thought this was easy, there's [a lot of easy pickings](https://bugzilla.mozilla.org/buglist.cgi?quicksearch=[good%20first%20bug&list_id=1316913) for Firefox and other Mozilla stuff. You can learn a bit about C++ and XUL, put a well known project on your CV, and most importantly feel great about yourself after doing something good for everyone... what is there more to ask for? :)

## ⁂
All in all Mozilla's [excellent documentation](https://developer.mozilla.org/) has made all of this an easy ride. You could wish more projects were documented like this. Additionally getting shown some tricks in *gdb* by *newsham* at 3 AM was great too — thanks!

One chap from the Moz team ended up [reblogging](http://blog.mozilla.com/dolske/2011/09/18/speaking-of-community/) this — pretty cool to consider they were just now talking about how to improve the provess I described here. Apparently [they are still thinking how to make things better](https://bugzilla.mozilla.org/show_bug.cgi?id=686998). From a business perspective it makes even more sense — you can get new workers up to date very quickly if the thing they will be working on is well-documented. This is yet more important for development teams that are geographically separated and even in very different time-zones.

This expands to the bigger picture of the [approachability of a project](http://blog.mozilla.com/dolske/2011/09/06/community-participation/) by the community, whether [open](http://cheater.posterous.com/automating-e2fsck) or [closed](http://notch.tumblr.com/post/9896830082/you-know-whats-fun) source. This is one of the things I hadn't seen Spolsky post on. Joel, your developers [are](http://www.joelonsoftware.com/items/2011/05/26.html) your [users](http://http://www.joelonsoftware.com/articles/BuildingCommunitieswithSo.html) too.

> The biggest annoyance is that they have a [policy](http://stackoverflow.com/questions/14293/are-there-any-unhappy-users-of-fogbugz/545990#545990) of not telling anyone what they are working on.

A free, open-source software has also got the benefit of the [converse](http://en.wikipedia.org/wiki/Contraposition) — their users are their developers. It brings the fact that the user can change the software much more to their liking; unless you make [warded-off](http://en.wikipedia.org/wiki/Application_programming_interface) [playgrounds](http://en.wikipedia.org/wiki/Plug-in_%28computing%29) you cannot have that in a closed-source application, although it [can](http://en.wikipedia.org/wiki/Virtual_Studio_Technology) be [very prolific](http://www.kvraudio.com/get.php?mode=results&st=adv&soft%5B%5D=i&soft%5B%5D=e&soft%5B%5D=h&soft%5B%5D=d&soft%5B%5D=w&type%5B%5D=0&f%5B%5D=0&f%5B%5D=au&f%5B%5D=dx&f%5B%5D=ladspa&f%5B%5D=rtas&f%5B%5D=vst&linux=1&osx=1&win=1&free=1&com=1&un=1&sf=0&receptor=&de=0&sort=3&rpp=100). What openness takes away is your ability to sell software for money, but [other things still seem to work](http://www.ecademy.com/node.php?id=164554). Only money buys you [food](http://www.fogcreek.com/fogbugz/), though, and I don't know of any way that can work for a small developer who wants to make his software completely open, and yet make money with it. Releasing source code without unit tests, APIs, and documentation does not count. You are left with things like [SaaS](http://www.gnu.org/philosophy/who-does-that-server-really-serve.html); until GNU and the FOSS community can come up with a business model for me I will not be able to make my software completely free. This might need [changes](http://chomsky.info/articles/20110824.htm) in how the whole society [works](http://en.wikipedia.org/wiki/Ubuntu_%28philosophy%29), but you know overdoing this [can](http://en.wikipedia.org/wiki/Vladimir_Lenin) sometimes [lead](http://en.wikipedia.org/wiki/Vyacheslav_Molotov) to [bad](http://en.wikipedia.org/wiki/Joseph_Stalin) things.

</markdown>
