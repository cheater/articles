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

## ⁂
All in all Mozilla's [excellent documentation](https://developer.mozilla.org/) has made all of this an easy ride. You could wish more projects were documented like this. Additionally getting shown some tricks in *gdb* by *newsham* at 3 AM was great too — thanks!

</markdown>
