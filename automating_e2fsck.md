<p><markdown>
###TLDR: this can be used as a guide to automating e2fsck and other e2tools programs when doing data recovery.

*note: please excuse the stupid display of code, there might be some errors because currently Posterous seems to be parsing the code as markdown. I have already emailed [Garry, the author of the feature](http://news.ycombinator.com/item?id=366732) about it, let's see what happens.*

I have recently been recovering a failed partition on my hard drive using e2fsck, and, after pressing 'y' for the billionth time I have remembered that "anything can be fixed by adding another layer of abstraction".

###The bash solution

First I have set up debugfs with some standard tools that would tell me if an inode is empty, and if yes, then I would go with the answer 'y'. Basically I created an empty dir, and in debugfs I have done *lcd* to change to that directory. Then I would type *dump &lt;inodeno&gt; inodeno* for each inode that e2fsck asked me about, and finally in a third terminal I would do a oneliner of [code]file *; strings * | less; rm -f *[/code]. This got boring pretty soon so I have decided to speed it up with some scripting. In #bash, resident badass [pgas](http://pgas.freeshell.org) has suggested named pipes to build it, so I did, and it looked sort of like this:

[code]
mkfifo dbfspipe
debugfs /dev/sdb2 &lt; dbfspipe
[/code]

and in another terminal:

[code]
while true; do
  echo -n '" '; read;
  echo "dump &lt;"$REPLY"&gt; "'"'$PWD/$REPLY'"' &gt; ../dbfspipe;
  sleep 2;
  echo $'\cc' &gt; ../dbfspipe;
  echo &gt; ../dbfspipe;
  sleep 0.5;
  file *;
  file * &gt;&gt; ../2.\ e2fsck\ progress.txt;
  rm -vf *;
  done
[/code]

The *$'\\cc'* is how you echo Ctrl-C in bash.  This worked &amp;em; however, every time the echo was done the named pipe closed and debugfs exited.  Again #bash to the rescue (thanks pgas!), it seems I can use a construct like the two loops below so that debugfs never sees the pipe closing:

[code]
while :; do
  while read; do
    printf %s\\n "$REPLY";
    done &lt; dbfspipe ;
  done | debugfs /dev/sdb2
[/code]

So, e2fsck would ask me about an inode, I would copy it over to the prompt in the *"while true"* loop above, press &lt;CR&gt;, and it would usually let me know the file was empty and that was my cue to press 'y'.

###That didn't really work

After the thousandth time, that was simply not good enough. I have made plans to automate the thing with Python, pexpect, and a lot of duct tape. In their infinite wisdom someone in the #ext4 channel (don't go there with usage questions please) told me that I should instead try and patch e2fsck, and even dug up an old patch which did something similar. So, being a bit scared about messing up e2fsck badly, I checked out the sources (from Sourceforge since kernel.org was down &amp;em; wow, Sourceforge, blast from the past!) and decided to hack around a bit. After using sulfuric acid (actually, several C related IRC channels &amp;em; same, right?) to unrust my C skills, and gawking through the code I have made some progress. [Eduard_Munteanu](https://plus.google.com/118385182586219825610/posts) on Freenode held my hand with C during that time.

###The C solution

As it turns out, all imperative languages are the same, even Haskell which has nothing on [functional languages like C](http://conal.net/blog/posts/the-c-language-is-purely-functional). The right place to hack was thus:

[code]
//e2fsck/util.c:
int ask (e2fsck_t ctx, const char * string, int def)
{               
        if (ctx-&gt;options &amp; E2F_OPT_NO) {
                printf (_("%s? no\n\n"), string);
                return 0;
        }
        if (ctx-&gt;options &amp; E2F_OPT_YES) {
                printf (_("%s? yes\n\n"), string);
                return 1;
        }
        if (ctx-&gt;options &amp; E2F_OPT_PREEN) {
                printf ("%s? %s\n\n", string, def ? _("yes") : _("no"));
                return def;
        }
        return ask_yn(string, def);
}
[/code]

I thought, alright, a question that answers itself! The *E2F_OPT_\** things are all applied to the options in *unix.c* with the help of *getopt(3)*. I decided I should replicate the logic behind *-p* (*E2F_OPT_PREEN*) or *-y* (*E2F_OPT_YES*) for my purposes: if a question is asked about a block of size 0, then just say yes.

The information was not available to me in this function, but it was available in its caller:

[code]
//e2fsck/problem.c
int fix_problem(e2fsck_t ctx, problem_t code, struct problem_context *pctx)
{
// ...
                } else
                        answer = ask(ctx, (ptr-&gt;prompt == PROMPT_NULL) ? "" :
                                     _(prompt[(int) ptr-&gt;prompt]), def_yn);
// ...
}
[/code]

### Setting up Vim with ctags and cscope
#
I tried grepping around for struct definitions but that was a failure. Apparently Ack is better but I haven't tried it; after people suggested I use ctags and cscope with [my vim](https://bitbucket.org/cheater/vimrc) was I able to figure out where the data I needed lies.

To configure them I used [this Stack Overflow guide](http://stackoverflow.com/questions/934233/cscope-or-ctags-why-choose-one-over-the-other). After installing the packages (*sudo aptitude install ctags cscope*) I went to the source root and ran *ctags* to build its database:

[code]
ctags -R
[/code]

and similarly I used the guide for setting up cscope, although I used a pipe because I don't like files lying around cluttering my bytes:

[code]
find . -name '*.py'  \
-o -name '*.java'    \
-o -iname '*.[CH]'   \
-o -name '*.cpp'     \
-o -name '*.cc'      \
-o -name '*.hpp' | cscope -b -q -i -
[/code]

After that was done, I was able to hover the name of a type and do *Ctrl-]* or *g]* in order to find the definition. So we have found:

[code]
//e2fsck/problem.h:
struct problem_context {
        errcode_t       errcode;
        ext2_ino_t ino, ino2, dir;
        struct ext2_inode *inode;
        struct ext2_dir_entry *dirent;
        blk64_t blk, blk2;
        e2_blkcnt_t     blkcount; 
        int             group;
        __u64   num;    
        const char *str;
};      
[/code]

[code]
//lib/ext2fs/ext2_fs.h:
struct ext2_inode {
        __u16   i_mode;         /* File mode */
        __u16   i_uid;          /* Low 16 bits of Owner Uid */
        __u32   i_size;         /* Size in bytes */
        __u32   i_atime;        /* Access time */
        __u32   i_ctime;        /* Inode change time */
        __u32   i_mtime;        /* Modification time */
        __u32   i_dtime;        /* Deletion Time */
        __u16   i_gid;          /* Low 16 bits of Group Id */
        __u16   i_links_count;  /* Links count */
        __u32   i_blocks;       /* Blocks count */
        __u32   i_flags;        /* File flags */
// ...
};
[/code]

It turned out what I want should be sitting inside *pctx-&gt;inode-&gt;i_size*. I have modified the *ask()* function adding a new parameter. However, *ask* is being used everywhere, so instead I made the old function a wrapper around the new one:

[code]
int ask (e2fsck_t ctx, const char * string, int def)
{
        return ask_skip (ctx, string, def, 0);
}                       
                                
int ask_skip (e2fsck_t ctx, const char * string, int def, int skip)
{                       
        if (ctx-&gt;options &amp; E2F_OPT_NO) {
                printf (_("%s? no\n\n"), string);
                return 0;       
        }               
        if (ctx-&gt;options &amp; E2F_OPT_YES) {
                printf (_("%s? yes\n\n"), string);
                return 1;
        }               
        if (skip || (ctx-&gt;options &amp; E2F_OPT_PREEN)) {
                printf ("%s? %s\n\n", string, def ? _("yes") : _("no"));
                return def;     
        }               
        return ask_yn(string, def);
}                               
[/code]

Then, I have modified the place in *fix_problem()* where *ask()* is called in the following way:

[code]
                } else {        
                        eff_yn = def_yn;
			skip = 0;
                        if ((pctx-&gt;inode != 0) &amp;&amp; (pctx-&gt;inode-&gt;i_size == 0)) {
                                skip = 1;
                                eff_yn = 1;
                        }
                        answer = ask_skip(ctx, (ptr-&gt;prompt == PROMPT_NULL) ?
                                          "" : _(prompt[(int) ptr-&gt;prompt]),
                                          eff_yn, skip);
                }       
[/code]

(of course, I also had to declare the new ints, *skip* and *eff_yn*, further above) After a quick *./configure* and *make* everything had, to my wonderment, compiled, and the check actually progressed automated like I wanted it to. However, then a lot of inodes started having other problems: bogus flags set, htree issues, dtime set on files that are not deleted, invalid fields, etc. The common thing was that eventually e2fsck also asked me if I want to truncate the inode to 0 length. This was fairly stupid &amp;em; I don't want to think about inodes which have nothing inside them anyways. They're not any of the host of important empty files in GNU/Linux you shouldn't touch, the inode numbers are real high and they're all probably in */home* somewhere, probably in the web cache for Firefox.

At first I have tried checking where *fix_problem()* is being called &amp;em; the order of checks seemed incorrect to me so I decoded to reorder them. However, after staring at the source for hours I have moved the size check:

[code]
//e2fsck/pass1.c
static void check_blocks(e2fsck_t ctx, struct problem_context *pctx,
                         char *block_buf)
{
// ...
        /* i_size for symlinks is checked elsewhere */
        if (bad_size &amp;&amp; !LINUX_S_ISLNK(inode-&gt;i_mode)) {
                pctx-&gt;num = (pb.last_block+1) * fs-&gt;blocksize;
                pctx-&gt;group = bad_size;
                if (fix_problem(ctx, PR_1_BAD_I_SIZE, pctx)) {
                        inode-&gt;i_size = pctx-&gt;num;
                        if (!LINUX_S_ISDIR(inode-&gt;i_mode))
                                inode-&gt;i_size_high = pctx-&gt;num &gt;&gt; 32;
                        dirty_inode++;
                }
                pctx-&gt;num = 0;
        }
// ...
}
[/code]

and the *if..eise* block supporting its behaviour all the way above other checks in *check_blocks()* and after a *make* I was greeted by e2fsck asking me if I want to truncate inode 2 from 4096 bytes to 0. Since inode 2 is '/' (always), I have decided I still don't know anything about the logic behind the check and backed out of it.

Eduard_Munteanu suggested a different approach: say no to all questions except for truncating inode sizes to 0. This was a great idea and I was able to do that in *fix_blocks()* using my previous strategy:

[code]
                } else {        
                        eff_yn = 0;
			skip = 1;
                        if ((pctx-&gt;inode != 0) &amp;&amp; (pctx-&gt;inode-&gt;i_size == 0)) {
                                skip = 1;
                                eff_yn = 1;
                        }
                        if ((code == PR_1_BAD_I_SIZE) &amp;&amp; (pctx-&gt;num == 0)) {
                                skip = 1;
                                eff_yn = 1;
                        }       
                        
                        answer = ask_skip(ctx, (ptr-&gt;prompt == PROMPT_NULL) ?
                                          "" : _(prompt[(int) ptr-&gt;prompt]),
                                          eff_yn, skip);
                }       
[/code]

From my earlier hacking of the caller of *fix_blocks* inside *pass1.c* I knew that the *code* for the question about changing size was *PR_1_BAD_I_SIZE* and that at this point the "problem context" variable *pctx* contained the *num* field, which was the size the inode would get resized to.

This worked quite well: fsck aborted all the fixes that I didn't want it to perform and it truncated all inodes it would have truncated to 0 anyways. I did a couple passes like this and then I started getting "legitimate" questions.

All in all this probably saved me days of sitting in front of the computer and pressing 'y', instead having taken days to figure out and modify the source :D

Afterwards, I have also noticed that some other questions kept cropping up, about flags and so on. This was easily alleviated by checking against the code:

[code]
                        if ((code == PR_1_COMPR_SET)
                          || (code == PR_1_HTREE_NODIR)
                          || (code == PR_1_SET_DTIME)
                          || (code == PR_1_EOFBLOCKS_FL_SET)
                          || (code == PR_1_EXTENTS_SET)
                          || (code == PR_1_EXTRA_ISIZE)
                          || (code == PR_1_ILLEGAL_BLOCK_NUM)
                          || (code == PR_1_SET_IMAGIC)) {
                                skip = 1;
                                eff_yn = 1;
                        }
[/code]

This seemed to do most of the work automatically. By now you might be asking yourself why I didn't just run *e2fsck -y*. Well, there are several reasons against doing so:

1. I don't know e2fsck well enough to know what fixes it might want to apply.  Any algorithm trying to automatically fix a system with unpredictable failure modes has to use heuristics. Heuristics can fail, and if it's your precious collection of bookmarks then you don't want to lose it.

1. I wanted to see how e2fsck goes about its business and what sorts of errors it finds. It is very useful to have a look inside the code for errors that you don't understand to see what's being done exactly. Some error messages in e2fsck are cryptic and require source traversal for "normal use"; in that case, it's helpful to search for the *PR_n_\** flags, since they only get used once in actual code and once in the huge list of all error messages in *problem.c*.

1. It's easier to keep track of what's going on if you can put the information into classes (as naturally dictated by the different *code* values I used) instead of having to look at a huge list of all sorts of different messages.

Finally, I have changed the logic to not automatically say no to further questions and have executed e2fsck in this way. All in all very few "real" questions have happened!

### Resources
Here are some resources and materials I have gathered when working with ext2:

1. [ext2 internal layout](http://www.nongnu.org/ext2-doc/ext2.html) by Dave Poirier

1. [another spec of ext2](http://uranus.chrysocome.net/explore2fs/es2fs.htm) by "John"

1. [Design and Implementation of the Second Extended Filesystem](http://web.mit.edu/tytso/www/linux/ext2intro.html) by Rémy Card, Theodore Ts'o, and Stephen Tweedie

1. [The extended-2 filesystem overview](http://www.virtualblueness.net/Ext2fs-overview/Ext2fs-overview-0.1.html) by Gadi Oxman

1. The #ext4 IRC channel on OFTC is very helpful. Please only go there only if you have questions that have to do with developing new tools for the ext2/3/4 file systems.

1. Similarly #linuxfs on the same place is for general Linux file system development, not only ext4

1. For questions regarding usage, data recovery, etc, ##linux on FreeNode is very helpful

1. Tools such as [GNU ddrescue](http://www.gnu.org/s/ddrescue/ddrescue.html), [ext2 debugfs](http://linux.die.net/man/8/debugfs) and [my FSCheck module for Python](https://bitbucket.org/cheater/fscheck/) can come in handy. Don't confuse ext2 debugfs with the linux kernel debugging interface also called debugfs.

1. [Evaluation of Linux ext2 file system debugger/debugfs for forensic use](http://computer-forensics.sans.org/community/papers/gcfa/eavaluation-linux-ext2-file-system-debugger-debugfs-forensic_83) by Michael Harvey

1. [The patch that was mentioned to me on #ext4](http://marc.info/?l=linux-ext4&m=118639926627598&w=4) - notice it has several bugs.

</markdown></p>
