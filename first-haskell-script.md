Title: How to create a Haskell Hello World script, read input, print output, and manipulate file contents
slug: first-haskell-script

<markdown>
Coming from an agile dev background, I like to do my development in scripts. I don't like the whole process of having to compile my stuff and run it; at least one step too many. Additionally things that are compiled are not portable. Here's how to create a simple script. This tutorial shows how to do terminal input and output, how to work with output buffering, how to read from and write to files, how to make loops in Haskell and use the *IO monad*. It assumes that you have [installed the Haskell Platform](http://hackage.haskell.org/platform/linux.html). If you're using [Windows](http://hackage.haskell.org/platform/), you might need to change some things (like how the file is interpreted). That part of the tutorial is mainly for people using GNU/Linux and similar systems, but the rest of it doesn't change at all because Haskell is very portable.

If you are having problems at any time, give the *#haskell* channel on Freenode a go — they're a very helpful bunch and the channel is active all the time. Feel free to drop comments here if something is unclear or just doesn't work.

## The whole shebang
The GNU/Linux kernel contains an interpreter. It's very simple and simple to write for, it is used for finding the right executor for the executable if it has a shebang. A countably infinite family of quines for the interpreter looks like this:
	#!/bin/cat
	(any other stuff here)
Haskell's interpreter for scripts (at least when using GHC) is called *runhaskell*. You might want to make the shebang *#!/usr/local/bin/runhaskell* but that would be wrong — the executable's location is not guaranteed, and, in fact, it changes loads because people often use a GHC which does not come from their package manager, and use some sort of prefix when installing. There's a trick to this:
	#!/usr/bin/env your-interpreter
The above will find the interpreter in *$PATH* and will execute it. So our file initially looks like this:
	#!/usr/bin/env runhaskell
## How to make it execute code
When *runhaskell* is passed a script it looks for the function *main* and executes it. The simplest *hello world* would then look like this:
	#!/usr/bin/env runhaskell
	main = putStrLn "Hello, World!"
Save that as *hello.hs*, *chmod a+x hello.hs* and you're good to go.

## Variables
This is the first departure of declarative languages (Haskell, ML) from constructive languages (C, Python). Haskell does not, in general, have variables as you know them from, say, Python. In constructive languages a variable is a *label* which points to a mutable (changeable) cell that points to a value. That's what the Python people mean when they say "Strings are immutable"; in Python:
	S = "Red"
	# now, S points to the place in memory where "Red" is stored.
	S = "Green"
	# S has stopped pointing to the place in memory where "Red" is
	# stored, and started pointing to the place in memory where
	# "Green" is stored. However, "Red" might still be stored if
	# there are other references to it. If S was the last label
	# for "Red", then "Red" gets dumped from memory.
In Haskell, you don't overwrite labels like in Python; you can, however, mask them:
	(some scope here)
	    (a block with some of the scope modified)
	    (more of that block here)
	    (yet more)
	(back to the old scope)
You can do that with *let*:
	-- on this line, x == "Red"
	putStrLn x           -- prints "Red"
	let x = "Green" in
	    putStrLn x       -- prints "Green"
	putStrLn x           -- prints "Red"
Variables in Haskell come from:

1. global definitions:
		x = "Hi"
	on a line of its own, unindented. This is the most basic scope in Haskell.
1. function arguments:
		hello x = "Hello, " ++ x ++ "!"
	This also applies to bindings in lambda expressions, *<-* inside *do*, etc.
1. *let* and friends:
		x = "Hello!"
		let x = "Good-bye!" in
		    putStrLn x
	This also applies to *case* and *where*.
 
## Getting input
Alright, let's make this thing interactive. You can read in stuff with *getLine*:
	main = getLine >>= putStrLn
The *>>=* there is a *method* of the *typeclass* called *Monad*. You don't need to understand all that, but what you need to get out of it is that *>>=* can mean different things in different contexts. In our context, we're talking about a specific *>>=* called "the *>>=* of *IO*". *IO*'s *>>=* works like this:
	new_action = action >>= callback
That is, *>>=* takes an *action*, executes it, plugs the output into the *callback*, and assigns what came out of that to *new_action*. The previous sentence is a lie.

## Working with programs here
You see, when doing *IO* stuff (that is, Haskell's input-output functionality), you are operating on *programs*. And out *action* is one such program which gets executed on runtime; this program can be meshed with other programs to generate other programs. Our line *new_action = action >>= callback* was not actually executing the program called *action*; it just mashed that program with the *callback*, without ever executing it. So, to be more concrete:
	main = getLine >>= putStrLn
the above code does not get a line and then print it; it creates a computer program which gets a line and then prints it. This program is then executed by our faithful *runhaskell*; only at this point do we actually get asked for input.

## Adding programs together
We know that *>>=* takes an action and a callback, and returns an action; this callback is not a *real program* in its own right; it lacks semantics. You can also execute two programs one after another using *>>*:
	main = (getLine >>= putStrLn) >> (getLine >>= putStrLn)
This takes the two programs called "*getLine >>= putStrLn*" and runs them one after another. Basically, *>>* works like this:
	new_action = old_action_1 >> old_action_2
The above definition of *main* will ask you for input, print it out, ask you for input, print it out, and then terminate.

## Building values
In Haskell, you create new values by relating them to other values. In Python, you create new values by taking old values and doing things with them. This is the most basic difference between Haskell and Python; or, if we talk about the families, you can say that this is the most basic difference between *declarative languages* (Haskell, ML, ...) and *constructive* languages (Python, C, ...).

A mathematician will understant this if I say this is like the difference between *synthetic* and *analytic* geometry. Indeed, in *synthetic* geometry you say:
	10    To get the bisector of a line segment,
	20    find the first point of the line segment and call it A
	30    find the second point of the line segment and call it B
	40    measure the distance between A and B and call it r
	50    draw a circle centered at A with radius r, call it C
	60    draw a circle centered at B with radius r, call it D
	70    find the points of intersection of C and D
	80    take one of those points of intersection, call it E
	90    take another one, call it F
	100   place a pencil on E
	110   place a straight edge so that it touches the pencil tip
	120   watching out that the straight edge stays adjacent to the tip, move it so that it also touches the point F
	130   draw a line against the straight edge
In Python, it could look like this:
	def line_bisector(line_segment):
	    """ Gets you a line bisector.
	        """
	    ends = line_segment.ends()
	    A = ends.pop()
	    B = ends.pop()
	    r = distance(A, B)
	    C = circle(A, r)
	    D = circle(B, r)
	    intersections = intersect_sets(C, D)
	    E = intersections.pop()
	    F = intersections.pop()
	    edge = StraightEdge()
	    edge.fix_at(E)
	    edge.fix_at(F)
	    edge.draw_line()
Not so in Haskell. In Haskell, you define *what things are*, rather than *how to get them*. In other words, you *declare* rather than *construct*. A Haskell version of this program could look like this in pseudocode:
	lineSisector line_segment = lineThrough(
	    intersectionOf(
	        circlesAtEnds(
	            line_segment
	            (lenghtOf line_segment)
	            )))
	
	circlesAtEnds line_segment = circlesAtPoints(endsOf(line_segment))
	-- endsOf is just an example, it does not get talked about just like
	-- line_segment.ends() did not get talked about in the Python code
	
	circlesAtPoints (x, y) = (circleAtPoint x, circleAtPoint y)
	
	lenghtOf line_segment = distance (fst line_segment) (snd line_segment)
	
	-- it is intuitive what circleAtPoint does, and what intersectionOf and
	-- lineThrough do; they are not explained in the Python code, either

This looks very similar to what you would say in analytic geometry:
	L = l(⋂ {S(P, |line_segment|) : P an end of line_segment})
	
	l({x_n}) = line going through x_n for all defined n
	
	⋂ {E_n} = {x : ∀ n, x ∊ E_n }
	
	|segment| = d(x, y) where x and y are ends of segment
Even without understanding the notation behind analytic geometry, you can look at the Haskell pseudocode above and notice that we have always defined things *in terms of other things*, without *doing* stuff with those other things. See it as the difference between an engineer making a blueprint of how a house should be, and the construction workers laying cement, bricks, and mortar. In the end, as a result, we get a house; the second way is much more long-winded and not really necessary just to demonstrate a house; a blueprint is good for our purposes.

## Looping stuff
If we want to repeat this over and over we need something akin to a loop. I can only guess where the name *loop* came from in the stone-age of computing where you wrote your programs in stone tablets — or perforated cards and tape — you could stick the ends of your perforated tape together in order for your *calculating machine* to perform the same action over and over. I remember once tacking together pieces of paper and feeding them to a printer in a loop; the printer transport broke; I assume tape loops would have been a very difficult mechanical contraption which would therefore cost a lot of money and would therefore be very desirable and hailed as the advent of new computing.

To cut on cost you could learn how to *rewind* tape to the beginning; this is much simpler, since the tape does not need to be kept in a loop that requires tension adjustment and a holding cartridge and rollers; you just reverse the direction of the engine (*driver*) in your tape drive, and you run it until you somehow figure out you are back to the beginning of the tape. For example, you can detect this with a mechanical object that is at the beginning of the tape and touches a switch that says "this is it".

Contemporary computers are still operating on tape; it might be billions of times faster than the original, solid-state cache, but it's still tape. This is reflected by languages adhering to this tape logic. Loops in C and Python and Java and even PHP are best illustrated in the best of all such languages, BASIC:
	10    PRINT "HELLO"
	20    PRINT "I AM A LOOP ITERATION"
	30    PRINT "I AM DONE, HAVE A NICE DAY!"
	40    GOTO 10
Haskell does not have this concept (it has something different, maybe better). Sure, you can have loops in Haskell just like you can have recursion in Python; it's advised against. It's unnatural, and, in the case of Python, it just plain doesn't work. It is unnatural in the same way that using English words when speaking German sounds stupid; in the same way that wearing trousers backwards doesn't work out. You *can* if you want to, but it's still stupid.

Haskell's idea of "things are compositions of other things" works again; instead of a loop instruction, in haskell we define something that repeats itself in this way:
	something that repeats is some job plus something that repeats
Or, to use pseudocode:
	something_that_repeats = something + something_that_repeats
Above, the *something* is our iteration; if we unroll this once, we get:
	something_that_repeats = something + something + something_that_repeats
If we unroll this again, we get:
	something_that_repeats = something + something + something + something_that_repeats
And so on. Here are some examples:
	shout = "A" ++ shout
	-- this gives us an infinite string: "AAAAAAAAAA..."
	
	rising x = [x] ++ rising (x+1)
	-- rising 1 will give us an infinite list: [1, 2, 3, ...]
	
	shout2 = "A" ++ shout2 ++ "A"
	-- this gives us the string faster ;-)
	
	shout3 = "AAAAAAA" ++ shout3 ++ "AAAAAAAAAAAAAAA"
	-- max optimization ;-)
This is, as you probably know, called *recursion*.

## Looping our program
Let's look at our short Haskell application again:
	#!/usr/bin/env runhaskell
	main = (getLine >>= putStrLn) >> (getLine >>= putStrLn)
In the same way, we can make our program recursive; just replace the second action on the right of *>>* with the whole thing:
	#!/usr/bin/env runhaskell
	main = (getLine >>= putStrLn) >> main
The above program will ask for input and print it out again, over and over, until we kill it.
	#!/usr/bin/env runhaskell
	main = (getLine >>= putStrLn) >> main >> main
The above will work too.

This will, however, not work:
	#!/usr/bin/env runhaskell
	main = main >> (getLine >>= putStrLn)
This is because, when constructing an infinite value (list, tree, etc), we should not recurse infinitely when defining one element of it; here, the first job that *main* would be doing is defined like that. This is called "head recursion".

This can be illustrated easier in this way:
	quiet = quiet ++ "A"
What is the first character of *quiet*? We don't really know.

We can also restructure our program to look more like a loop, in this way:
	iteration = getLine >>= putStrLn
	main = iteration >> main
Let us also comment on our code a bit; at 2 lines it might already be too complex for some to understand:
	iteration = getLine >>= putStrLn
	-- echoes what we type in
	
	main = iteration >> main
	-- The main program
Finally, we can introduce *do* notation; this is a special notation for *IO* which wraps around *>>=* and makes our *IO actions* more legible. This exposes *IO* for what it really is: simulation of a *constructive* language inside the *declarative* language Haskell. We use *do* like this:
	#!/usr/bin/env runhaskell
	iteration = do
	-- echoes what we type in
	    x <- getLine
	    putStrLn x
	
	main = iteration >> main
	-- The main program
Notice that we have assignments here with the *<-* operator. This is exactly equivalent to the previous version of *iteration*. The Freenode IRC channel *#haskell* has a very nice bot called *lambdabot* with a few useful commands. You can use them in the channel, or in a query with *lambdabot*. One such command is *@undo*, which we can use to unwrap do notation:
	@undo iteration = do x <- getLine; putStrLn x
	iteration = getLine >>= \x -> putStrLn x
Now to understand this you need to know a bit about lambda calculus. The short version is, that the function:
	f x = putStrLn x
can also be expressed as
	f = \x -> putStrLn x
Think of *\\*, (or *lambda*, or *𝜆* as it is called in other languages) as popping a value off the *argv* in C. Now the above is exactly equivalent to:
	f = putStrLn
since the two functions take the same kinds and numbers of arguments.

## More functionality
We might now want the program to do some more things; for example, let's have it greet people:
	#!/usr/bin/env runhaskell
	iteration = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    name <- getLine
	    putStr ("Hello, " ++ name ++ ". ")
	    putStrLn "Pleased to meet you!"
	
	main = iteration >> main
	-- The main program
This will naturally echo out greetings. The *putStr* function prints out a string, and *putStrLn* prints it out and moves to a new line. Let's give it a prompt; we want the line on which we will be entering to start with "> ":
	#!/usr/bin/env runhaskell
	iteration = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    name <- getLine
	    putStr ("Hello, " ++ name ++ ". ")
	    putStrLn "Pleased to meet you!"
	
	main = iteration >> main
	-- The main program
Oops! This doesn't quite work as expected. The output we get is:
	$ ./guess.hs 
	Please enter your name...
	Bob
	> Hello, Bob. Pleased to meet you!
	Please enter your name...
Some of you might have figured out that Haskell only flushes output when a new line is printed; we need to explicitly flush the output with *hFlush*. It is, however, not loaded at startup; we must import the module it is in, *System.IO*:
	import System.IO
This makes our namespace messy, though; it dumps all the stuff that's inside System.IO directly into the global context and if we try to go back to source code that does that with umpteens of modules, then, after ten years (or ten weeks), we can't figure out what came from where. Instead, let's use a qualifier:
	import qualified System.IO as I
Now we can use *hFlush*. It takes the stream to flush as a parameter; *System.IO.stdout* is one such stream.
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	iteration = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    putStr ("Hello, " ++ name ++ ". ")
	    putStrLn "Pleased to meet you!"
	
	main = iteration >> main
	-- The main program
This works perfectly!

Now, let's make this script react to certain people. First, let's define some names:
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
Then, let's make a function that checks if the person is known:
	known name = any (== name) known_friends
The above requires some explanation. First of all, the syntax may be confusing. The first word on the line is *known*, the name of the new value (new function). Then come the names of the parameters, if any. Then we have an equals sign (*=*); then comes the code, the body of, the function. Second, *any* is a function that takes a comparison and a list, and returns a logical value. The comparison should take one parameter, and return a logical value. Third, *(== name)* is a comparison function just like we need; in Haskell, everthing is a function, and so is *==*; however, *==* is a function of two parameters, and our comparison needs to take one parameter and evaluate to a truth value. We can manipulate functions using *partial application* (also known as *currying*) in order to get functions of less variables; this way we got from the function *==* which takes two variables to a function which just takes one variable and is called *== name*. If we wanted to compare with just one name, we could have also written it like this:
	isPhilippa name = "Philippa" == name
	-- tells us whether a name is equal to "Philippa"
	philippaKnown = any isPhilippa known_friends
However, we would want to parametrize:
	isSomeone someone name = someone == name
Then, we need to partially apply the function:
	isPhilippa name = isSomeone "Philippa" name
What we have just done is to create a function called *isPhilippa*, which is *isSomeone* with its first parameter, *someone*, fixed to the value *"Philippa"*. However, this is just the same as:
	isPhilippa = isSomeone "Philippa"
The above version uses so called "point free" syntax; it is used in mathematics to define functions in terms of other functions without actually passing around the point at which those two functions interface. In other words, we can write something like:
	ctg = 1/tan
Which has the benefit that, if we use notation saying that e.g. *r* is a real number and *z* is a complex number, then we'd have to define the functions first for real numbers:
	ctg r = 1/(tan r)
...and then for complex numbers, once we figure out how to extend our trigonometric functions to the complex plane:
	ctg z = 1/(tan z)
This looks stupid because it's the same thing. Point-free notation avoids this sort of situation.

Now we could use something like that:
	isSomeone someone name = someone == name
	isPhilippa = isSomeone "Philippa"
	philippaKnown = any isPhilippa known_friends
However, this is just the same as:
	isSomeone someone name = someone == name
	philippaKnown = any (isSomeone "Philippa") known_friends
Now, we can rewrite the definition of *isSomeone*:
	isSomeone someone name = (==) someone name
	philippaKnown = any (isSomeone "Philippa") known_friends
This changes *==* from being an *infix operator* to being a normal function. We can now remove a parameter:
	isSomeone someone = (==) someone
And then another one:
	isSomeone = (==)
...so in the end, we have:
	philippaKnown = any ((==) "Philippa") known_friends
Now we can parametrize the name:
	known name = any ((==) name) known_friends
or just:
	known name = any (== name) known_friends

Note that, technically, the last line above fixes the second argument, whereas the lines before that fixed the first parameter of *==*. Makes no difference this time around, though.

OK, let's go back to our program. We want it to check if he knows the person in question; and we want it to change the greeting. Here's what we have written until now:
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name = any (== name) known_friends
	-- Tells us if a person is known
	
	iteration = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    putStr ("Hello, " ++ name ++ ". ")
	    putStrLn "Pleased to meet you!"
	
	main = iteration >> main
	-- The main program
Time to add an *if* clause:
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name = any (== name) known_friends
	-- Tells us if a person is known
	
	iteration = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name
	        then putStrLn ("Hello, " ++ name ++ "! How have you been?")
	        else putStrLn ("Pleased to meet you, " ++ name ++ "!")
	
	main = iteration >> main
	-- The main program

## Remembering people — how to *return* from a *do* block
In order for *iteration* to remember the people it is greeting, we need to be able to change what list of friends it operates on. You cannot append to lists in Haskell. You can, however, pass in different lists of friends. First, let's make the *friends* get passed around in a loop. Let's change:
	main = iteration >> main
to:
	loop friends = iteration >> loop friends
	-- Loops the whole thing.
	
	main = loop known_friends
	-- The main program
Now we need to make *iteration* take *friends* as a parameter, which we will pass down to *known*.
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name friends = any (== name) friends
	-- Tells us if a person is known
	
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then putStrLn ("Hello, " ++ name ++ "! How have you been?")
	        else putStrLn ("Pleased to meet you, " ++ name ++ "!")
	
	loop friends = iteration friends >> loop friends
	-- Loops the whole thing.
	
	main = loop known_friends
	-- The main program
Then, we need to make *iteration* yield a value. Let's first change our loop to *do* notation before we do that:
	loop friends = do
	-- Loops the whole thing.
	    iteration friends
	    loop friends
	main = loop known_friends
	-- The main program
Now, let's have *iteration* yield a value. The return value of a *do* block is the result of the last action in it; we can just put a getLine at the end of our *iteration*:
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then putStrLn ("Hello, " ++ name ++ "! How have you been?")
	        else putStrLn ("Pleased to meet you, " ++ name ++ "!")
	    putStrLn "Please enter a name to remember as a friend..."
...and then use the result of that and append it to our list of friends:
	loop friends = do
	-- Loops the whole thing.
	    new_friend <- iteration friends
	    loop ([new_friend] ++ friends)
However, Haskell has a special operator for prepending to lists, strings, etc that we can use:
	    loop (new_friend:friends)

We might just want our program to remember people who it greeted; we can't do that right now because we get the name of the person greeted at the top of the *iteration*, but it needs to be the result of the last line of the *do* block! We therefore need to somehow take that result, called *name*, and make it the result of another computation. For this, we can use the *return* function. It's a method of every *Monad*; in the *IO* monad it's kinda like a *noop* or an *identity function* in that it takes a computation result, and makes it the result of the computation we are defining. Thus, we can do:
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then putStrLn ("Hello, " ++ name ++ "! How have you been?")
	        else putStrLn ("Pleased to meet you, " ++ name ++ "!")
	    return name
...and our script can now remember us!

## Using *Just* as out-of-band communication
Right now our script can add names to the list; however it will add duplicates. We want to prevent this. We could use a sentinel value; if that shows up we don't add it. For example: 
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return ""
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return name
	
	loop friends = do
	-- Loops the whole thing.
	    new_friend <- iteration friends
	    if new_friend == ""
	        then loop friends
	        else loop (new_friend:friends)
Notice two things. Firstly, we are using *return* inside *then* and *else*. The value being *return*ed percolates up to the *if* and becomes its value; since the *if* is the last thing in our *do* block, it's the return value of the *do* block.

Secondly, notice that we had to wrap the bodies of the *then* and *else* in *do* blocks themselves. The body of a *then* or *else* must be a single value; that value is the resultant value of the whole *if* clause, depending on what branch is chosen. A *do* block is just a single value; *do* composes multiple values together and exposes the one on the last line as its value.

So, if we are in the *else*, our *return name* bubbles up to the *do* it's immediately inside, then to the *else* that *do* is inside, then to the *if*. Since the *if* is the last ting in the big *do* block, that *return name* bubbles up to that, and finally becomes the return value of *iteration*.

To see that everything works the way we imagine it does we can use the *print* function. It takes any value and prints out a representation of it to *stdout*; it's useful for debugging.
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    if new_friend == ""
	        then loop friends
	        else loop (new_friend:friends)
Here is our whole program until now:
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name friends = any (== name) friends
	-- Tells us if a person is known
	
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return ""
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return name
	
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    if new_friend == ""
	        then loop friends
	        else loop (new_friend:friends)
	
	main = loop known_friends
	-- The main program

What we are doing here is called "in-band signaling"; this is a term which comes from the world of radio, information theory, and communication. It means selecting a special value of our type, and using it with additional semantics. Naturally, no one will have a name which consists of the empty string; However, it's not so great to use in-band signaling if our values are less predictable; for example, how do we use in-band signaling when returning a number? We could use some huge number, but that breaks when someone uses that specific number; bad idea. Additionally, other programmers who use our function might not be aware of those special semantics; this is summed up by stating that the semantics of a value should be encoded in its type. That is, a number is a number, and a string is a sting, and that's all. Full stop. Finito.

To do this properly, we can use *out-of-band signaling*. In some languages we could throw an exception; Haskell can do that, but let's not do that just yet. We could also return a pair, where the first element is the status, and the second element is the actual value being returned. That's stupid, though; what do we return as the actual value if we don't want to return anything? The implementation of our type's semantics could leak out to the user code of the users still keep on relying on *that* as a sentinel value. Finally, many languages allow you to return *None*, *Null*, or *False* in place of the string; it's a form of polymorphism. Haskell's functions can only return one type; if they return a string in one place, it has to be a string everywhere else too or the program won't even compile. The good thing is we can create new types in Haskell. Kinda how classes in many object oriented languages end up being used, except Haskell types don't have the inheritance drama; additionally, they're just as good as *str* in Python or *int* in C++ and aren't second-category citizens like classes in those languages are. One way to create a special type is using the keyword *Maybe*.

Before we go on, let's transform our *loop* function a bit:
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        "" -> loop friends
	        _ -> loop (new_friend:friends)
The *case* clause is like a cooler version of *if*; you know it from some *constructive* languages as well. It allows us to have multiple branches and match against values; additionally we can do *pattern matching*. The first case which is matched is used; further cases are not inspected. The final option has a pattern of *_*. A pattern of the form *x* just matches everything. It is used as a catch-all in case the previous options didn't match.

Now let's use *Maybe* to do some out-of-band signaling. There are two out-of-band values for *Maybe*: *Just* and *Nothing*. You use *Just* by making your return value *Just whatever_value*; you use *Nothing* by simply making your return value *Nothing*. Using *Nothing* is much like using *Null* in Java, *None* in Python or *0* or *-1* in C, [but without the problems](http://qconlondon.com/london-2009/presentation/Null+References%3A+The+Billion+Dollar+Mistake). Let's convert *iteration*; our current version is:
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return ""
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return name
We change it to:
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return Nothing
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return (Just name)
Our signal has changed, so we need to change not only the place we emit it, but also where we receive it:
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends
	        Just new_friend -> loop (new_friend:friends)
Notice two cool things. Firstly, we barely changed anything, but this change gives the compiler the ability to reason about our code in a much more advanced way. Secondly, notice that this time we have made a more involved pattern match; it's *Just new_friend*. It gives us direct access to *new_friend* as a variable. You can match against all sorts of structures which is very powerful; this allows you to extract values that are buried deep inside the value you are unpacking.
Here is our whole program until now:
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name friends = any (== name) friends
	-- Tells us if a person is known
	
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return Nothing
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return (Just name)
	
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends
	        Just new_friend -> loop (new_friend:friends)
	
	main = loop known_friends
	-- The main program

## Persistence using *IO* — reading from and writing to files
So we can make our script learn new friends! However, every time it terminates its brain is reset. Let's try storing that information somewhere. But first, we need to be able to *read* it. You may know Python's *with* keyword; it creates a new context within which e.g. a file handle is available. Haskell has something similar for files; it's called *withFile* and, as everything, it is a function. Instead of creating a block or context, *withFile* takes a callback as an argument, which it then executes with the handle as the parameter to that callback. Before we apply it, let's transform our current code a bit; change:
	main = loop known_friends
	-- The main program
to:
	main = do
	-- The main program
	    friends <- return known_friends
	    loop friends
Not really a change at all. We are still passing *known_friends* to *loop*. If you don't know why we're using *return* here, read above where we introduce *return* for the first time. Let's split the initialization into a separate function:
	start = do
	-- Initializes the program
	    friends <- return known_friends
	    loop friends
	
	main = start
	-- The main program
Trivial. Now, however, *start* can act as our callback. We get *withFile* from *System.IO* which we already import in our script.
	start conf = do
	-- Initializes the program
	    friends <- return known_friends
	    loop friends
	
	main = I.withFile "friends.conf" I.ReadWriteMode start
	-- The main program
It's easy to see that the first parameter is the file name, the second is the opening mode (kinda like in *fopen(3)*, see *man fopen*), and the third is the callback that makes use of the new file handle. Let's get the contents:
	start conf = do
	-- Initializes the program
	    contents <- I.hGetContents conf
	    print contents
	    friends <- return known_friends
	    loop friends
We are printing the *contents* for debugging purposes. Seems to be working! Try it with a file yourself. Ideally, we would first rewind the handle, because our function doesn't really know where the handle comes from; it might be dirty; you don't know where that handle was.
	start conf = do
	-- Initializes the program
	    I.hSeek conf I.AbsoluteSeek 0
	    contents <- I.hGetContents conf
	    print contents
	    friends <- return known_friends
	    loop friends
Now, let's use the contents of the file:
	start conf = do
	-- Initializes the program
	    I.hSeek conf I.AbsoluteSeek 0
	    contents <- I.hGetContents conf
	    print contents
	    friends <- return (lines contents)
	    loop friends
We have used the *lines* function, which takes a string, and splits it on newlines into a list of strings. This is exactly what we want. We can simplify our code:
	start conf = do
	-- Initializes the program
	    I.hSeek conf I.AbsoluteSeek 0
	    contents <- I.hGetContents conf
	    print contents
	    loop (lines contents)
OK, so our program is configurable now, but it doesn't store data yet. Let's get about to doing that now. We need to write to our handle, and the best place to do that is inside *loop*, since that's where we figure out if a name is new or not. We first need to pass the parameter:

in *start*, change:
	    loop (lines contents)
to:
	    loop (lines contents) conf
and in *loop*, change:
	loop friends = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends
	        Just new_friend -> loop (new_friend:friends)
to:
	loop friends conf = do
	-- Loops the whole thing.
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends conf
	        Just new_friend -> loop (new_friend:friends) conf
We also need to position the handle correctly; we want it to be at the end, and we don't want to depend on the semantics of side-effects:
	loop friends conf = do
	-- Loops the whole thing.
	    I.hSeek conf I.SeekFromEnd 0
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends conf
	        Just new_friend -> loop (new_friend:friends) conf
Uh-oh, this doesn't work! [As it turns out](http://hackage.haskell.org/packages/archive/base/latest/doc/html/System-IO.html#v:hGetContents), the handle is closed already at this point; this happened in *hGetContents*; but we want to keep appending. We need to figure out a different way to get the contents. Let's write a function which takes a handle, and returns all the lines in a list:
	parseConf conf = do
	    -- Parses the config file given a handle.
	    eof <- I.hIsEOF conf
	    if eof
	        then return []
	        else do
	            line <- I.hGetLine conf
	            rest <- parseConf conf
	            return (line:rest)
Let's also make *start* use the thing:
	start conf = do
	-- Initializes the program
	    I.hSeek conf I.AbsoluteSeek 0
	    friends <- parseConf conf
	    print friends
	    loop friends conf
That works! Let's go back to hacking *loop*: we just need to write to the handle and we should be good.
	loop friends conf = do
	-- Loops the whole thing.
	    I.hSeek conf I.SeekFromEnd 0
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends conf
	        Just new_friend -> do
	            I.hPutStrLn conf new_friend
	            loop (new_friend:friends) conf
Give it a try — it works! The program writes to the file and reads back from it when restarted; its digital identity is persistent. Here is the whole program as it should be:
	#!/usr/bin/env runhaskell
	import qualified System.IO as I
	
	known_friends = ["Don", "Simon", "Eduard"]
	-- This program knows those people
	
	known name friends = any (== name) friends
	-- Tells us if a person is known
	
	iteration friends = do
	    -- Greets someone
	    putStrLn "Please enter your name..."
	    putStr "> "
	    I.hFlush I.stdout
	    name <- getLine
	    if known name friends
	        then do
	            putStrLn ("Hello, " ++ name ++ "! How have you been?")
	            return Nothing
	        else do
	            putStrLn ("Pleased to meet you, " ++ name ++ "!")
	            return (Just name)
	
	parseConf conf = do
	    -- Parses the config file given a handle.
	    eof <- I.hIsEOF conf
	    if eof
	        then return []
	        else do
	            line <- I.hGetLine conf
	            rest <- parseConf conf
	            return (line:rest)
	
	loop friends conf = do
	-- Loops the whole thing.
	    I.hSeek conf I.SeekFromEnd 0
	    print friends
	    new_friend <- iteration friends
	    case new_friend of
	        Nothing -> loop friends conf
	        Just new_friend -> do
	            I.hPutStrLn conf new_friend
	            loop (new_friend:friends) conf
	
	start conf = do
	-- Initializes the program
	    I.hSeek conf I.AbsoluteSeek 0
	    friends <- parseConf conf
	    print friends
	    loop friends conf
	
	main = I.withFile "friends.conf" I.ReadWriteMode start
	-- The main program

# ⁂
We have learnt how to get keyboard input and do terminal output, we have learnt how to read from and write to files. This should get you guys started doing some practical things with Haskell as a general systems language. It's very powerful for that, especially given how portable it is. I hope you liked this tutorial; there are probably errors in it — if you notice any, please post! This tutorial might get changed a bit or enhanced if new ideas come up.

The Haskell community (specifically the *#haskell* IRC channel on Freenode) has been very helpful in creating this tutorial; they have at some point explained literally every single Haskell thing in this document! Really, there's little more you could wish for; those guys rule.
</markdown>
