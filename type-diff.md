Title: Idea: type-diff, a tool for managing code in Haskell and other type-inferred languages
slug: type-diff
<markdown>
I have just stumbled upon [Michael Feathers' blog](http://michaelfeathers.typepad.com). If you haven't read any of Michael's stuff then let me assure you it is [very good reading](http://www.amazon.com/Working-Effectively-Legacy-Michael-Feathers/dp/0131177052).

In one post, [Michael makes a point about type inference and dynamic typing](http://michaelfeathers.typepad.com/michael_feathers_blog/2010/12/the-fertile-middle-ground-between-static-and-dynamic-typing.html):

> I came across [a blog post] this morning.  It describes research into inferring types from production runs of dynamically typed programs. [...]
> Anything we can do to make programming easier is good.  In particular, typing type annotations seems suspect if we can generate them from runtime data.  We can avoid premature design commitment and still reap benefits by post-annotating code in an IDE or generating on the fly static checkers.

This got me creative. Here's my response in the comments:

> Michael, I like the idea of using production runs to type code. However, here's the thing: why do we assign types to our code? "To know what it does" breaks up into two sub-reasons that I can identify:
>
> 1. when first writing code, it helps speed up development and makes it easy to define the general structure of our idea; if our structure sucks then the type checker will find out
> 2. when maintaining code, it helps ensure that no one does something stupid that would change the way a lot of our code works. An example is if we have a program which accepts data, processes it to strings, and then does a lot of manipulation on those strings; then prints them.
>
> block of data -> process into strings -> work on strings -> print string output
>
> For example, calculate the frequency of strings, do some heavy stats on them, and then print them out.
>
> At some point we have to fix the fact that in some huge file the back end crashes and change the input back end to spit out (Int, String)to get the line number that string was on in the file. Instead of String and now half our application is not passing around (Int, String) and the front-end doesn't accept that anymore for printing.
>
> In this situation you would see that the front-end is not adhering to a specific type and you might be compelled to change the front end; this is not the whole story however and the fix is probably to change the back end, given that the processor doesn't care about the line number anyways.
>
> This can be aided by some sort of meta SCM with knowledge of the language it's storing; it could tell you how many of the inferred types have changed between two revisions; if it's, statistically seen, too much (noticeably more than the average) then it could mean trouble; probably something you might want to fix.
>
> The SCM would need to understand the structure of the language and keep track of the types. Not trivial -- but not too hard if you're into this sort of thing. A lot of people write parsers in "FP" languages so "FP" languages with type inference are more likely to get this sort of solution.

Does anyone else think this would be a good idea for boosting Haskell productivity? Right now people use types as unit tests; in fact the most repeated mantra is to stringently describe types of stuff that your module exports as a public API, but not to bother with doing that for the innards. I think that's wrong — on the one hand, fixing the API with types limits reuse and with restrictive typing you can break things rather than fix things; on the other hand, you need to keep track when the "private" code breaks, as well. When it does break you're usually looking for a specific commit or series of commits in your SCM that will have messed it up. Often, broken code means broken types are inferred. Such a diff, a *type-diff* kind of tool could be used to track down where this has happened; you could use the size of the delta coming from a *type-diff* tool as an interesting code metric.

Regarding strong typing of public APIs — it is my experience that giving more power to users is better than walling them in. Give the users the ability to influence the type being inferred; but give them an ability to figure out what breaks, too. With a *type-diff* kind of tool you could have both: the ability to make the types more flexible, but also the ability to see when incorrect use of your code messes up the types in that third-party module you're using.
</markdown>
