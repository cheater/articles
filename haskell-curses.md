Title: Using Haskell to easily build interactive text-mode applications with HSCurses, NCurses or Vty
slug: haskell-curses
<markdown>
I have recently been musing on writing some more involved programs in Haskell that would require a *curses(3)* based UI, and decided to check out the available libraries. There's a deficit in documentation on how to do stuff with them, so I thought I'd share.

This article creates a program with an interactive user interface that displays a list of things, kind of like in an email program or a file manager; it displays a "cursor" by highlighting the line it's on, and allows you to move the selection. When the selection moves to the edge of the screen it scrolls the list.

The same program is developed for *HSCurses*, *NCurses* and *Vty*. In-depth explanations are in the *HSCurses* part, the *NCurses* and *Vty* parts just compare the differences. Currently, *Vty* looks like the best choice; followed not-so-closely by *NCurses*. The *HSCurses* library did not perform well enough to suggest its use.

The description of every library has a section called "Caveat emptor" which describes the problems encountered with it; *Vty* does not have any.

## Available libraries
There are three notable packages for Haskell:

1. *[hscurses](http://hackage.haskell.org/package/hscurses)* which is fairly established and works well. It's somewhat low-level at times, but then that makes it very similar to how you use *curses* in *C*-like languages
1. *[ncurses](http://hackage.haskell.org/package/ncurses)* — newer than *hscurses* and somewhat higher-level. One downside for me was not being able to run it in interpreted mode, it displayed garbage unless I compiled my program and ran the binary. Might be the best choice if you want absolute cursor positioning and incremental updates; I'm not sure how you can do this well with *vty*.
1. *[vty](http://hackage.haskell.org/package/vty)*, not as new as Haskell's *ncurses*; a complete rewrite in Haskell, without using the *ncurses(3)* family of libraries. Good for things where *curses* is not available; does not work on [Windows](http://mjg59.dreamwidth.org/5552.html). The best choice in my opinion for being so insanely easy to create complex UIs with.

## *HSCurses*
Let's take a look at HSCurses first. There's a [pretty good tutorial for HSCurses in Japanese](http://d.hatena.ne.jp/itchyny/20110902/1314965062). You can also get an example package called *[hscurses-fish-ex](http://hackage.haskell.org/package/hscurses-fish-ex)* which displays a fish tank and is a sort of tech demo; you can find its source under *~/.cabal/packages/hackage.haskell.org/hscurses-fish-ex/*.

You can install the package via *cabal install hscurses* and the tutorial via *cabal install hscurses-fish-ex*.

If you untar the source for *hscurses-fish-ex* then you will find everything is in one big file that can be executed as a script with *./hscurses-fish-ex.hs*. Good for experimentation; you can edit the file in *Vim* and after writing do *:!./%* to execute it. It's a bit too complicated for a first approach, though. Let's go back to [itchyny](http://twitter.com/#!/itchyny)'s excellent guide.

You can also learn some tricks from [Michael Forster's post about *HSCurses* on Haskell Cafe](http://www.mail-archive.com/haskell-cafe@haskell.org/msg83946.html) from 2010.

## Hello, *HSCurses*
On his "Programming Mumblings" ("プログラムモグモグ") blog, itchyny starts with something similar to this:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	
	main :: IO ()
	main = do
	    HSCurses.initCurses
	        -- initCurses :: IO ()
	        -- 初期化する
	    HSCurses.wAddStr HSCurses.stdScr "Hello, HSCurses"
	        -- wAddStr :: Window -> String -> IO ()
	        -- 標準の画面に文字を出力
	    HSCurses.refresh
	        -- refresh :: IO ()
	        -- refreshすることが必要
	    HSCurses.getCh
	        -- getCh :: IO Key
	        -- 一文字入力を待つ
	    HSCurses.endWin
	        -- endWin :: IO ()
	        -- Cursesを用いたアプリケーションを
	        -- 終了するときは, これが必要

I guess it translates to something like:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	
	main :: IO ()
	main = do
	-- prints a welcome message
	    HSCurses.initCurses
	        -- initCurses :: IO ()
	        -- starts up the Curses: puts the terminal in
	        -- graphical mode, moves the cursor, etc.
	    HSCurses.wAddStr HSCurses.stdScr "Hello, HSCurses"
	        -- wAddStr :: Window -> String -> IO ()
	        -- prints a string :) to the screen in the first
	        -- parameter. HSCurses.stdScr is the standard
	        -- Window that gets created; you can create more
	        -- windows, too, and print to them separately.
	    HSCurses.refresh
	        -- refresh :: IO ()
	        -- refreshes the screen.
	    HSCurses.getCh
	        -- getCh :: IO Key
	        -- waits for input
	    HSCurses.endWin
	        -- endWin :: IO ()
	        -- frees all the resources

I like the minimalism of the examples in that blog post. This one doesn't even use managed resources for simplicity; if someone aborts before endWin is called they could crash their terminal. The author rectifies that soon using *bracket_*.

The *bracket_* function is similar to Python's *with* and takes three functions that it executes:
	bracket_ allocate deallocate work

First it runs *allocate*, then runs the *work* function, then runs *deallocate*. The deallocator is also executed when *work* exits in some abnormal way. You can think of *work* as the managed context that Python's *with* creates.

With *bracket_*, we would rewrite our code to:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	import qualified Control.Exception as Exception
	
	allocate = do
	-- allocates resources
	    HSCurses.initCurses
	
	deallocate = do
	-- frees resources
	    HSCurses.endWin
	
	work = do
	-- prints a welcome message
	    HSCurses.wAddStr HSCurses.stdScr "Hello, HSCurses"
	    HSCurses.refresh
	    HSCurses.getCh
	    return ()
	
	main :: IO ()
	main = Exception.bracket_ allocate deallocate work 
	-- the main program

Note that now we can also *return* meaningful values from *work* which was previously impossible because *endWin* took that possibility. We get the bonus that our allocator and deallocator can be next to eachother in the source file, making it easier to spot any discrepancies between the two which could result in bugs.

Let's get our program more modular; how about we display five messages? In order to display a message at a specific position, you can use *mvWAddStr*:
	HSCurses.mvWAddStr HSCurses.stdScr 6 2 "Hello, HSCurses"

The above puts the "H" in "Hello" on the third column of the seventh line. Let's modularize a bit:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	import qualified Control.Exception as Exception
	import Control.Monad (forM_)
	
	allocate = do
	-- allocates resources
	    HSCurses.initCurses
	
	deallocate = do
	-- frees resources
	    HSCurses.endWin
	
	printHello line col = do
	-- prints a welcome message
	    HSCurses.mvWAddStr HSCurses.stdScr line col "Hello, HSCurses"
	
	work = do
	-- displays welcome messages
	    forM_ [0..5] $ \line ->
	        printHello line 0
	    HSCurses.refresh
	    HSCurses.getCh
	    return ()
	
	main :: IO ()
	main = Exception.bracket_ allocate deallocate work 
	-- the main program

Here we are using *forM_* to iterate over a *List*; it's just like Python's *for ... in* statement:
	forM_ ourList $ \x ->
	    myFunc x

is just like Python's:
	for x in our_list:
	    my_func(x)

They can be nested, too:
	forM_ ourList $ \x ->
	    forM_ ourList2 $ \y ->
	        myFunc x y

or, in Python:
	for x in our_list:
	    for y in our_list_2:
	        my_func(x, y)

We are using the *mvWAddStr* function in order to print the strings. You have to be careful with it — if your printing offset is beyond the screen then the program crashes; if the string sticks out past the screen then you get wrapping which messes up your display. It's an idea to use *mvAddCh* which doesn't have the crashing problem; however it still wraps and additionally doesn't let us specify the window. Let's write a wrapper for mvWAddStr:
	mvWAddStr2 w y x s = do
	-- a safe alternative to mvWAddStr
	    (rows, cols) <- HSCurses.scrSize
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = cols - x - 1
	        let s2 = take space s
	        HSCurses.mvWAddStr w y x s2

Getting the off-by-one errors out took some testing; the first version of this that I tried would crash, only exactly if the horizontal offset was equal to the screen width. Here we are using *when* as an *if* without the *else*. The *when* function comes from *Control.Monad*.

Let's apply the new function to *printHello*:
	printHello line col = do
	-- prints a welcome message
	    mvWAddStr2 HSCurses.stdScr line col "Hello, HSCurses"

## Let's make a list of things
Alright, now let's try displaying something more meaningful. How about a list of cheeses:
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]

Let's also add some supporting functions:
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.

	printCheese line cheese = do
	-- prints out info on a cheese
	    mvWAddStr2 HSCurses.stdScr line 0 $ getName cheese

and finally let's make use of this in the application:
	work = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese line cheese
	    HSCurses.refresh
	    HSCurses.getCh
	    return ()

Our program now displays cheese. Let's now make the thing interactive :)

## Interactivity
In order to have interactivity we first need the ability to mark where we are. Let's give the display a cursor:
	printCheese line cheese cursor = do
	-- prints out info on a cheese
	    let indicator = if cursor
	        then " > "
	        else "   "
	    mvWAddStr2 HSCurses.stdScr line 0 $ indicator ++ getName cheese

Additionally, we need to change *work*:
	        printCheese line cheese False

Now, let's make the program respond to keyboard input. It already responds to input by quitting, let's specify that a bit more. The *getCh* function returns all sorts of events; we're only interested in *KeyChar* events:
	work = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese line cheese False
	    HSCurses.refresh
	    kc <- HSCurses.getCh
	    case kc of
	        HSCurses.KeyChar c -> handleKeyboard c
	        _ -> return ()

The idea here is that *handleKeyboard* will call *work* again:
	handleKeyboard c = case c of
	-- handles keyboard input
	    'q' -> return ()
	    _ -> work

The code quits on *Q* and continues on anything else. Let's add the cursor and make it react to input:

in *work*:
	work requestedPosition = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese line cheese (line == position)

in *handleKeyboard*:
	handleKeyboard c position = case c of
	-- handles keyboard input
	    'q' -> return ()
	    _ -> work ((position + 1) `mod` 10)

in *main*:
	main = Exception.bracket_ allocate deallocate (work 0)

Make the cursor actually respond like we want it to:
	handleKeyboard c position = case c of
	-- handles keyboard input
	    'q' -> return ()
	    'j' -> work (position + 1)
	    'k' -> work (position - 1)
	    _ -> work position

Let's add the ability to scroll. In order to do that we need to be able to display the list with an offset:
	handleKeyboard c position offset = case c of
	(...)
	
	work requestedPosition offset = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese (line + offset) cheese (line == position)
	    HSCurses.refresh
	    kc <- HSCurses.getCh
	    case kc of
	        HSCurses.KeyChar c -> handleKeyboard c position offset
	        _ -> return ()
	
	main = Exception.bracket_ allocate deallocate (work 0 0)

Passing *(-10)* as *offset* from *main* showed the list scrolled down — a proof that our scheme works. Let's make the view scroll with the cursor:
	work requestedPosition offset = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    (rows, cols) <- HSCurses.scrSize
	    let screenPosition = position + offset
	    let offset2 = if screenPosition >= rows
	        then offset - (screenPosition - rows + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese (line + offset2) cheese (line == position)
	    HSCurses.refresh
	    kc <- HSCurses.getCh
	    case kc of
	        HSCurses.KeyChar c -> handleKeyboard c position offset2
	        _ -> return ()

This seems to be working, except it doesn't blank — so we get garbage where the screen isn't being refreshed. Let's call a blanker:
	    HSCurses.wclear HSCurses.stdScr
	    let fromages2 = zip [0..] fromages

Works great!

## Colors
Let's make the line the cursor is on colorize. Terminals display text with foreground and background colors. We can use the module *UI.HSCurses.CursesHelper* to help ourselves:
	import qualified UI.HSCurses.CursesHelper as HSCursesHelper

We have to define a new *color pair*, which is conceptually an object describing the foreground and background color of a glyph on the terminal. We can do it in *allocate*:
	allocate = do
	-- allocates resources
	    HSCurses.initCurses
	    hasColors <- HSCurses.hasColors
	    if hasColors
	        then do
	            HSCurses.startColor
	            HSCurses.initPair
	                (HSCurses.Pair 1)
	                (HSCursesHelper.black)
	                (HSCursesHelper.white)
	            return ()
	        else
	            return ()

We are calling *hasColors* to branch and then are setting a new color pair with *initPair*. The syntax is:
	HSCurses.initPair (HSCurses.Pair pairNum) (foregroundColor) (backgroundColor)

Now to use the new *color pair* we modify *printCheese*:
	printCheese line cheese cursor = do
	-- prints out info on a cheese
	    let (indicator, colorPair) = if cursor
	        then (" > ", 1)
	        else ("   ", 0)
	    HSCurses.attrSet HSCurses.attr0 (HSCurses.Pair colorPair)
	    mvWAddStr2 HSCurses.stdScr line 0 $ indicator ++ getName cheese

Works great! Let's make the line span the width of the terminal:
	printCheese line cheese cursor = do
	-- prints out info on a cheese
	    let (indicator, colorPair) = if cursor
	        then (" > ", 1)
	        else ("   ", 0)
	    HSCurses.attrSet HSCurses.attr0 (HSCurses.Pair colorPair)
	    (rows, cols) <- HSCurses.scrSize
	    mvWAddStr2 HSCurses.stdScr line 0 $ replicate cols ' ' 
	    mvWAddStr2 HSCurses.stdScr line 0 $ indicator ++ getName cheese 

The program should look like this right now:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	import qualified UI.HSCurses.CursesHelper as HSCursesHelper
	import qualified Control.Exception as Exception
	import Control.Monad (forM_, when)
	
	allocate = do
	-- allocates resources
	    HSCurses.initCurses
	    hasColors <- HSCurses.hasColors
	    if hasColors
	        then do
	            HSCurses.startColor
	            HSCurses.initPair
	                (HSCurses.Pair 1)
	                (HSCursesHelper.black)
	                (HSCursesHelper.white)
	            return ()
	        else
	            return ()
	
	deallocate = do
	-- frees resources
	    HSCurses.endWin
	
	mvWAddStr2 w y x s = do
	-- a safe alternative to mvWAddStr
	    (rows, cols) <- HSCurses.scrSize
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = cols - x - 1
	        let s2 = take space s
	        HSCurses.mvWAddStr w y x s2
	
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]
	
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.
	
	printCheese line cheese cursor = do
	-- prints out info on a cheese
	    let (indicator, colorPair) = if cursor
	        then (" > ", 1)
	        else ("   ", 0)
	    HSCurses.attrSet HSCurses.attr0 (HSCurses.Pair colorPair)
	    (rows, cols) <- HSCurses.scrSize
	    mvWAddStr2 HSCurses.stdScr line 0 $ replicate cols ' ' 
	    mvWAddStr2 HSCurses.stdScr line 0 $ indicator ++ getName cheese 
	
	handleKeyboard c position offset = case c of
	-- handles keyboard input
	    'q' -> return ()
	    'j' -> work (position + 1) offset
	    'k' -> work (position - 1) offset
	    _ -> work position offset
	
	work requestedPosition offset = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    (rows, cols) <- HSCurses.scrSize
	    let screenPosition = position + offset
	    let offset2 = if screenPosition >= rows
	        then offset - (screenPosition - rows + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    HSCurses.wclear HSCurses.stdScr
	    let fromages2 = zip [0..] fromages
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese (line + offset2) cheese (line == position)
	    HSCurses.refresh
	    kc <- HSCurses.getCh
	    case kc of
	        HSCurses.KeyChar c -> handleKeyboard c position offset2
	        _ -> return ()
	
	main :: IO ()
	main = Exception.bracket_ allocate deallocate (work 0 0)
	-- the main program

## Caveat emptor
There is one problem: the "safe" string printing function we have made is not so safe; it breaks on multibyte characters: they pass through Haskell as a single character, but when *HSCurses* gets to them it counts them as multiple characters, and prints them as some sort of escape codes. This can crash your application. The alternative is to use *mvAddCh* instead; this however breaks things such as selecting words on the terminal, and does not let you print to a specific window. The *mvwaddch* function is not exported by *HSCurses*, either. I tried compiling the program and that did not help, either. By the virtue of these issues I see *HSCurses* as broken and not usable in stable programs.

## *NCurses*
In order to install the *ncurses* package you need to have *c2hs* installed. Apparently *cabal* will take care of dependencies on libraries, but will not install new parsers/compilers/etc. You can install *c2hs* using *cabal* as well. A bit weird, but ok.

Another issue with *NCurses* is that it doesn't seem to work in interpreted mode — if you can get it to run, please tell me how! For me, it displays garbage unless the program using it is compiled.

*NCurses* uses *Data.Text* for its strings, which is a *better* solution than just using Haskell strings. It could matter if you try to store 50 000 000 lines of text in memory.

When writing the *HSCurses*-based script, I have been doing *:! %* in *Vim* in order to execute the script. The *%* evaluates to the file name. This time, you have to compile it and execute the binary. The new shorthand is *:!ghc % && /path/to/binary-name*. For example, I am editing this as */tmp/nc.hs*, so I execute *:!ghc % && /tmp/nc* every time I want to run the program's newest version.

## Hello, *NCurses*
The simplest script in *NCurses* is similar to the one in *HSCurses*:
	module Main where
	
	import qualified UI.NCurses as NCurses
	import qualified Data.Text as Text
	
	main = NCurses.runCurses $ do
	-- prints a greeting
	    win <- NCurses.defaultWindow
	    NCurses.updateWindow win $ do
	        NCurses.drawText $ Text.pack "Hello again, NCurses!"
	    NCurses.render
	    NCurses.getEvent win Nothing

This is very simple. The *main* function consists of very discernible things:
	main = NCurses.runCurses $ do
	        -- you need to call runCurses to execute your IO. Additionally,
	        -- runCurses acts like somewhat of a resource manager.
	-- prints a greeting
	    win <- NCurses.defaultWindow
	        -- get the default window. Just like HSCurses.
	    NCurses.updateWindow win $ do
	        -- update a window through the actions passed
	        NCurses.drawText $ Text.pack "Hello again, NCurses!"
	        -- action to draw text at the current cursor position.
	    NCurses.render
	        -- update the terminal with new window contents.
	    NCurses.getEvent win Nothing
	        -- read key (or something else, like a mouse click or event)

Compare with *HSCurses*:
	#!/usr/bin/env runhaskell
	import qualified UI.HSCurses.Curses as HSCurses
	
	main :: IO ()
	main = do
	-- prints a welcome message
	    HSCurses.initCurses
	        -- initCurses :: IO ()
	        -- starts up the Curses: puts the terminal in
	        -- graphical mode, moves the cursor, etc.
	    HSCurses.wAddStr HSCurses.stdScr "Hello, HSCurses"
	        -- wAddStr :: Window -> String -> IO ()
	        -- prints a string :) to the screen in the first
	        -- parameter. HSCurses.stdScr is the standard
	        -- Window that gets created; you can create more
	        -- windows, too, and print to them separately.
	    HSCurses.refresh
	        -- refresh :: IO ()
	        -- refreshes the screen.
	    HSCurses.getCh
	        -- getCh :: IO Key
	        -- waits for input
	    HSCurses.endWin
	        -- endWin :: IO ()
	        -- frees all the resources

Those things are pretty much identical in size (if you strip out the explanations). This comparison is, however, irrelevant to actual use, because the *HSCurses* code above is broken either way — for lack of resource management. *NCurses* wins here.

We are capturing quite a bit with that *getEvent*. Let's make it only capture key presses:
	readChar win = do
	-- captures keyboard input
	    event <- NCurses.getEvent win Nothing
	    case event of
	        Just (NCurses.EventCharacter _) -> return ()
	        _ -> readChar win
	
	main = NCurses.runCurses $ do
	-- prints a greeting
	    win <- NCurses.defaultWindow
	    NCurses.updateWindow win $ do
	        NCurses.drawText $ Text.pack "Hello again, NCurses!"
	    NCurses.render
	    readChar win

No need to separate out into a *work* like function, since it's pretty much all of *main*, but let's do it for clarity and ease of comparison:
	work = do
	-- prints a greeting
	    win <- NCurses.defaultWindow
	    NCurses.updateWindow win $ do
	        NCurses.drawText $ Text.pack "Hello again, NCurses!"
	    NCurses.render
	    readChar win
	
	main = NCurses.runCurses $ work
	-- the main program

Now, let's make *NCurses* print the output five times. In *HSCurses* we did something like this:
	printHello line col = do
	-- prints a welcome message
	    HSCurses.mvWAddStr HSCurses.stdScr line col "Hello, HSCurses"
	
	work = do
	-- displays welcome messages
	    forM_ [0..5] $ \line ->
	        printHello line 0
	    HSCurses.refresh
	    HSCurses.getCh
	    return ()

*NCurses* does not have a special function to print text at an offset; instead, it has *moveCursor* which explicitly allows you to move the cursor; afterwards you can print using *drawText*:
	printHello line col = do
	-- prints a welcome message
	    NCurses.moveCursor line col
	    NCurses.drawText $ Text.pack "Hello again, Curses!"
	
	work = do
	-- displays welcome messages
	    win <- NCurses.defaultWindow
	    NCurses.updateWindow win $ do
	        forM_ [0..5] $ \line ->
	            printHello line 0
	    NCurses.render
	    readChar win

Here we are again using the *forM_* loop construct.

The *drawText* and *moveCursor* combo in *NCurses* has the same problems as *mvWAddStr* in *HSCurses*: try to move the cursor outside of your screen and you crash; try to print something that'll wrap and you destroy your screen. Let's try to wrap these like with *HSCurses*; as a bonus we can get rid of *Text.pack* and *NCurses.updateWindow*:
	drawTextAt win y' x' s = do
	-- a safe alternative to moveCursor and drawText
	    (rows, cols) <- NCurses.screenSize
	    let y = toInteger y'
	    let x = toInteger x'
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = fromInteger $ cols - x - 1
	        let s2 = take space s
	        NCurses.updateWindow win $ do
	            NCurses.moveCursor (toInteger y) (toInteger x)
	            NCurses.drawText $ Text.pack s2

Compare with:
	mvWAddStr2 w y x s = do
	-- a safe alternative to mvWAddStr
	    (rows, cols) <- HSCurses.scrSize
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = cols - x - 1
	        let s2 = take space s
	        HSCurses.mvWAddStr w y x s2

The two are pretty much the same: instead of *HSCurses.scrSize* you call *NCurses.screenSize*; you call *NCurses.updateWindow*; and you have to juggle the types a little bit, because *NCurses* was written by some sort of type freak. Personally I haven't seen a terminal with more than *536870911* rows or columns, and that's the age-old Haskell '98 bound on *Int*.

Of course, we have to change the other functions a bit as well to pass the *win* parameter. Here is the whole source as it is right now:
	module Main where
	
	import qualified UI.NCurses as NCurses
	import qualified Data.Text as Text
	import Control.Monad (forM_, when)
	
	readChar win = do
	-- captures keyboard input
	    event <- NCurses.getEvent win Nothing
	    case event of
	        Just (NCurses.EventCharacter _) -> return ()
	        _ -> readChar win
	
	drawTextAt win y x s = do
	-- a safe alternative to moveCursor and drawText
	    (rows, cols) <- NCurses.screenSize
	    let y = toInteger y'
	    let x = toInteger x'
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = fromInteger $ cols - x - 1
	        let s2 = take space s
	        NCurses.updateWindow win $ do
	            NCurses.moveCursor (toInteger y) (toInteger x)
	            NCurses.drawText $ Text.pack s2
	
	printHello win line col = do
	-- prints a welcome message
	    drawTextAt win line col "Hello again, Curses!"
	
	work = do
	-- displays welcome messages
	    win <- NCurses.defaultWindow
	    forM_ [0..5] $ \line ->
	        printHello win line 0
	    NCurses.render
	    readChar win
	
	main = NCurses.runCurses $ work
	-- the main program

## Let's make another list of things
Let's have more cheese:
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]

The *getName* function has not changed:
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.

whereas *printCheese* changes only slightly:
	printCheese win line cheese = do
	-- prints out info on a cheese
	    drawTextAt win line 0 $ getName cheese

and finally *work* only has cosmetical differences from its *HSCurses* counterpart:
	work = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win line cheese
	    NCurses.render
	    readChar win

The really nice thing here is that multibyte characters *work*, inlike in *HSCurses*. This is a big plus in my eyes.

## Interactivity
Making the *NCourses* version of our program is trivial, just like in *HSCourses*. The changes are exactly the same and not worth describing again, here's the code:
	printCheese win line cheese cursor = do
	-- prints out info on a cheese
	    let indicator = if cursor
	        then " > "
	        else "   "
	    drawTextAt win line 0 $ indicator ++ getName cheese

in *work*:
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win line cheese False

Making the program respond to keyboard input is very similar as well:
	handleKeyboard c = case c of
	-- handles keyboard input
	    'q' -> return ()
	    _ -> work

	work = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win line cheese False
	    NCurses.render
	    ev <- NCurses.getEvent win Nothing
	    case ev of
	        Just (NCurses.EventCharacter c) -> handleKeyboard c
	        _ -> return ()

here we can throw away *readChar*; it's just a copy-paste to gut it and get the familiar layout from *HSCurses*. The notable difference here is that we get the input from *NCurses.getEvent win Nothing* and that we match against a different pattern. The *handleKeyboard* function is exactly the same.

Implementing our round-robin functionality is exactly the same; I'll spare you the explanations and show you the new code:
	handleKeyboard c position = case c of
	-- handles keyboard input
	    'q' -> return ()
	    _ -> work ((position + 1) `mod` 10)
	
	work requestedPosition = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win line cheese (line == position)
	    NCurses.render
	    ev <- NCurses.getEvent win Nothing
	    case ev of
	        Just (NCurses.EventCharacter c) -> handleKeyboard c position
	        _ -> return ()
	
	main = NCurses.runCurses $ (work 0)
	-- the main program

Giving control over the cursor is *exactly* the same:
	handleKeyboard c position = case c of
	-- handles keyboard input
	    'q' -> return ()
	    'j' -> work (position + 1)
	    'k' -> work (position - 1)
	    _ -> work position

as is adding offset functionality:
	handleKeyboard c position offset = case c of
	(...)
	
	work requestedPosition offset = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win (line + offset) cheese (line == position)
	    NCurses.render
	    ev <- NCurses.getEvent win Nothing
	    case ev of
	        Just (NCurses.EventCharacter c) -> handleKeyboard c position offset
	        _ -> return ()
	
	main = NCurses.runCurses $ (work 0 0)
	-- the main program

and making the view scroll with the cursor:
	work requestedPosition offset = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    (rows, cols) <- NCurses.screenSize
	    let screenPosition = position + offset
	    let offset2 = if (toInteger screenPosition) >= rows
	        then fromInteger $
	            (toInteger offset) - ((toInteger screenPosition) - rows + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win (line + offset2) cheese (line == position)
	    NCurses.render
	    ev <- NCurses.getEvent win Nothing
	    case ev of
	        Just (NCurses.EventCharacter c) -> handleKeyboard c position offset2
	        _ -> return ()

Blanking is fairly easy; you just call *NCurses.render* in *main*:
	    win <- NCurses.defaultWindow
	    NCurses.render
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win (line + offset2) cheese (line == position)

It works just as expected.

## Colors
Setting up colors is somewhat similar. There are still pairs of colors and they are still assigned to integer ID's. Everything is in the *NSCurses* module, no need to import new things.

Creating an allocator works differently; *NSCurses* does not use side-effects like *HSCurses* does; since *work* is run by the monad we can use *bind* in order to pass it parameters:
	main = NCurses.runCurses $ allocate >>= (work 0 0)
	-- the main program

The allocator is very simple; we can leverage *Maybe* to take care of the case when the terminal is monochromatic; we did not do that with *HSCurses* although it's not impossible to imagine something similar. The basic thing you want to do is:
	allocate = do
	-- sets up NCurses.
	    NCurses.newColorID NCurses.ColorWhite NCurses.ColorBlack 1

With conditioning it looks like this:
	allocate = do
	-- sets up NCurses.
	    maxId <- NCurses.maxColorID
	    if  maxId > 0
	        then do
	            supportsColor <- NCurses.supportsColor
	            if supportsColor
	                then do
	                    cid <- (NCurses.newColorID
	                        NCurses.ColorBlack
	                        NCurses.ColorWhite
	                        1
	                        )
	                    return (Just cid)
	                else return Nothing
	        else return Nothing

The *work* and *handleKeyboard* functions only need to accept the new parameter for the thing to compile:
	handleKeyboard c position offset colorId = case c of
	(...)
	
	work requestedPosition offset colorId = do
	(...)

In order to make the display use the new color you need to pass it onto printCheese:
	printCheese win line cheese cursor colorId = do
	-- prints out info on a cheese
	    let (indicator, useColor) = if cursor
	        then (" > ", True)
	        else ("   ", False)
	    case colorId of
	        Just cid -> when useColor $ do
	            NCurses.updateWindow win $ NCurses.setColor cid
	        Nothing -> return ()
	    drawTextAt win line 0 $ indicator ++ getName cheese
	    NCurses.updateWindow win $ NCurses.setColor NCurses.defaultColorID

If we make the selection line as wide as the terminal the end of *printCheese* looks like this:
	    (rows, cols) <- NCurses.screenSize
	    drawTextAt win line 0 $ replicate (fromInteger cols) ' ' 
	    drawTextAt win line 0 $ indicator ++ getName cheese
	    NCurses.updateWindow win $ NCurses.setColor NCurses.defaultColorID

The code should currently look like this:
	module Main where
	
	import qualified UI.NCurses as NCurses
	import qualified Data.Text as Text
	import Control.Monad (forM_, when)
	
	allocate = do
	-- sets up NCurses.
	    maxId <- NCurses.maxColorID
	    if  maxId > 0
	        then do
	            supportsColor <- NCurses.supportsColor
	            if supportsColor
	                then do
	                    cid <- (NCurses.newColorID
	                        NCurses.ColorBlack
	                        NCurses.ColorWhite
	                        1
	                        )
	                    return (Just cid)
	                else return Nothing
	        else return Nothing
	
	drawTextAt win y' x' s = do
	-- a safe alternative to moveCursor and drawText
	    (rows, cols) <- NCurses.screenSize
	    let y = toInteger y'
	    let x = toInteger x'
	    when ((y >= 0) && (x >= 0) && (y < rows) && (x < cols)) $ do
	        let space = fromInteger $ cols - x - 1
	        let s2 = take space s
	        NCurses.updateWindow win $ do
	            NCurses.moveCursor (toInteger y) (toInteger x)
	            NCurses.drawText $ Text.pack s2
	
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]
	
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.
	        
	printCheese win line cheese cursor colorId = do
	-- prints out info on a cheese
	    let (indicator, useColor) = if cursor
	        then (" > ", True)
	        else ("   ", False)
	    case colorId of
	        Just cid -> when useColor $ do
	            NCurses.updateWindow win $ NCurses.setColor cid
	        Nothing -> return ()
	    (rows, cols) <- NCurses.screenSize
	    drawTextAt win line 0 $ replicate (fromInteger cols) ' ' 
	    drawTextAt win line 0 $ indicator ++ getName cheese
	    NCurses.updateWindow win $ NCurses.setColor NCurses.defaultColorID
	
	handleKeyboard c position offset colorId = case c of
	-- handles keyboard input
	    'q' -> return ()
	    'j' -> work (position + 1) offset colorId
	    'k' -> work (position - 1) offset colorId
	    _ -> work position offset colorId
	
	work requestedPosition offset colorId = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    (rows, cols) <- NCurses.screenSize
	    let screenPosition = position + offset
	    let offset2 = if (toInteger screenPosition) >= rows
	        then fromInteger $
	            (toInteger offset) - ((toInteger screenPosition) - rows + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    let fromages2 = zip [0..] fromages
	    win <- NCurses.defaultWindow
	    NCurses.render
	    forM_ fromages2 $ \(line, cheese) ->
	        printCheese win (line + offset2) cheese (line == position) colorId
	    NCurses.render
	    ev <- NCurses.getEvent win Nothing
	    case ev of
	        Just (NCurses.EventCharacter c) ->
	            handleKeyboard c position offset2 colorId
	        _ -> return ()
	
	main = NCurses.runCurses $ allocate >>= (work 0 0)
	-- the main program

## Caveat emptor
All in all *NCurses* was a much better experience than *HSCurses*. A more functional monadic shell coupled with a more concise set of functions has made this possible. The fact that it works with multibyte characters makes it a much better choice than *HSCurses*. There have been some quirks however: I have encountered some redrawing problems where the last row wasn't being updated if the view was scrolling down. One workaround is to ignore the last line and make all your code pretend the terminal is one row shorter; this is not a real fix however. I have been unable to fix this issue. If you have any ideas, please let me know. Even with the described problem *NCurses* is much better than *HSCurses*.

## *Vty*
The [vty package](http://hackage.haskell.org/package/vty) is a departure from the *curses* library and tries to do everything on its own. It's structured differently, into many small modules, and uses the concept of "images" and "pictures". Here is a [short description of Vty](http://www.haskell.org/haskellwiki/Library/VTY):
> A very simple terminal interface library.
> 
> In 150 non-blank non-comment lines of Haskell (and 7 lines of C) vty provides:
> 
> - Automatic handling of suspend/resume (SIGTSTP+SIGCONT)
> - Automatic handling of window resizes
> - Automatic computation of minimal differences
> - Minimizes repaint area, thus virtually eliminating the flicker problem that plagues ncurses programs
> - Automatically decodes keyboard keys into (key,[modifier]) tuples
> - Automatically supports refresh on Ctrl-L.
> - Automatically supports timeout after 50ms for lone ESC (a barely noticable delay)
> - Extensive color scheme support: background colors, default colors, reverse-video, bold, underline, half-bright, and blinking attributes.
> - Unicode characters on output, automatically setting and resetting UTF-8 mode (beware double width and combining characters!)
> - Disables ISIG and IXOFF, allowing C-q, C-s, C-c, C-z, and C-\ to be received as input.
> - Interface is designed for relatively easy compatible extension.
> 
> Current disadvantages:
> 
> - No current support for non-ANSI terminals.
> - Minimal support for special keys on terminals other than the linux-console. (F1-5 and arrow keys should work, but anything shifted isn't likely to.)
> - Uses the TIOCGWINSZ ioctl to find the current window size, which appears to be limited to Linux and BSD.

The *Vty* module suite is much easier and seems higher-level than *NCurses* or even *HSCurses*. It can be used to easily make complicated displays, but can also be used in a lower-level way if you want to use *putStrLn* and *hFlush*. I think it has to be very good for slightly upgrading the visual output of what otherwise is a listing in the terminal — think of colorizing grep output, making a spinner for wget, making progress bars, etc.

## Hello, *Vty*!
Vty has the concept of *Image*s and *Picture*s. Basically an *Image* seems to be the basic unit of graphics (think tiles / sprites), and a *Picture* is sort of like a finalized, rendered full-screen display (think Super Mario on NES).

Apparently, *Vty*'s idea of graphical output is like this: you use *Graphics.Vty.Picture.pic_for_image* to transform an *Image* into a *Picture*. Finally, you use *Graphics.Vty.update* to print to *Picture* to your terminal.

Let's make a "hello world" which puts the terminal in graphic mode and waits for input before terminating:
	import qualified Graphics.Vty as Vty
	
	main = do
	-- the main program
	    vt <- Vty.mkVty
	    t <- Vty.terminal_handle
	    putStrLn "Hello, Vty!"
	    Vty.next_event vt
	    Vty.release_terminal t

This code is much simpler than in the previous version of this tutorial. The author of *Vty* was very helpful and answered my emails on the same day as I have sent them, which helped substantially in getting the code into shape.

We won't move the cursor to print single lines. Instead, we'll make use of *Vty*'s ability to concatenate *Image* values.
For those who want to know, we can move the cursor with *set_cursor_pos*:
	    Vty.set_cursor_pos t 6 2
It is notable that the coordinates are swapped in comparison to *HSCurses* and *NCurses*.

*Vty*'s approach of concatenating *Image*s is a major departure from the menial task of moving the cursor and padding that we encountered in *HSCurses* and *NCurses*. I wonder if it's possible to do this in those, too. This approach is great; in a previous version of this blog post, I have been also using cursor-moving to print with *Vty*. After the author, Corey, tipped me off on concatenation, the complete code was reduced by one third, and was much easier and simpler to use — while performing faster and fixing the instability issues I have been experiencing before.

We will use the *vert_cat* function which takes a list of *Image*s and creates a single *Image* by gluing them one under another.
	main = do
	-- the main program
	    vt <- Vty.mkVty
	    let image = Vty.string Vty.current_attr "hello, Vty!"
	    let images = take 5 $ repeat image
	    let imagesUnified = Vty.vert_cat images
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    Vty.next_event vt
	    Vty.shutdown vt

No flushing or anything like that required. Just works!

## Let's again make a list of things
Everything is better with a bit of french cheese added:
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]

The supporting functions:
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.

	fromageImage cheese = do
	-- prints out info on a cheese
	    Vty.string Vty.current_attr $ getName cheese

Plug this into *main*:
	main = do
	-- the main program
	    vt <- Vty.mkVty
	    let fromages2 = zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese)
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    Vty.next_event vt
	    Vty.shutdown vt

Works! Additionally, multibyte characters are displayed as well, which is very good.

## Interactivity
In order to have interactivity we first need the ability to mark where we are. Let's give the display a cursor:
	fromageImage cheese cursor = do
	-- prints out info on a cheese
	    let indicator = if cursor
	        then " > "
	        else "   "
	    Vty.string Vty.current_attr $ indicator ++ (getName cheese)

Additionally, we need to change *main*:
	            (\(line, cheese) -> fromageImage cheese False)

Before we go on with the keyboard control we need to set up the allocator; the manual says we should only have one copy of *Vty.Vty* around, otherwise *Vty* will bug out. Additionally, we need to *shutdown* at the end. Let's do the usual:
	allocate = do
	-- sets up Vty
	    vt <- Vty.mkVty
	    return vt
	
	deallocate vt =
	-- frees Vty resouces
	    Vty.shutdown vt
	
	work vt = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese False)
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    Vty.next_event vt
	    return vt
	
	main = allocate >>= work >>= deallocate
	-- the main program

Actual keyboard control looks nearly the same as in *HSCurses* and *NCurses*; this time we pattern-match against *EvKey*, which has the nice ability to tell us about the keyboard event and the currently pressed modifiers separately: *EvKey Key [Modifier]*. The *Key* can again be pattern-matched against *KASCII c*. The *handleKeyboard* function is almost exactly the same; the only difference is that *work* has to ultimately return the terminal handle for the *deallocate* function to use:
	handleKeyboard c vt = case c of
	-- handles keyboard input
	    'q' -> return vt
	    _ -> work vt
	
	work vt = do
	-- displays cheeses
	    let fromages2 = zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese False)
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    ev <- Vty.next_event vt
	    case ev of
	        Vty.EvKey (Vty.KASCII c) [] -> handleKeyboard c vt
	        _ -> return vt

Now it's time to make the program display the cursor and respond to keyboard input:
	handleKeyboard c position vt = case c of
	-- handles keyboard input
	    'q' -> return vt
	    _ -> work ((position + 1) `mod` 10) vt
	
	work requestedPosition vt = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese (line == position))
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    ev <- Vty.next_event vt
	    case ev of
	        Vty.EvKey (Vty.KASCII c) [] -> handleKeyboard c position vt
	        _ -> return vt
	
	main = allocate >>= (work 0) >>= deallocate
	-- the main program

Making the cursor respond to *j* and *k* is the same, too:
	handleKeyboard c position vt = case c of
	-- handles keyboard input
	    'q' -> return vt
	    'j' -> work (position + 1) vt
	    'k' -> work (position - 1) vt
	    _ -> work position vt

Now it's time to add the ability to scroll. First, add the *offset*:
	handleKeyboard c position offset vt = case c of
	-- handles keyboard input
	    'q' -> return vt
	    'j' -> work (position + 1) offset vt
	    'k' -> work (position - 1) offset vt
	    _ -> work position offset vt
	
	work requestedPosition offset vt = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    let fromages2 = drop (0 - offset) $ zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese (line == position))
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    ev <- Vty.next_event vt
	    case ev of
	        Vty.EvKey (Vty.KASCII c) [] -> handleKeyboard c position offset vt
	        _ -> return vt
	
	main = allocate >>= (work 0 0) >>= deallocate
	-- the main program

You can plug in *(work 0 (-1))* to see the *offset* in action.

Then, the ability to move the view with the cursor is added as expected:
	work requestedPosition offset vt = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    Vty.DisplayRegion cols rows <- (Vty.terminal_handle >>= Vty.display_bounds)
	    let (cols2, rows2) = (fromEnum cols, fromEnum rows)
	    let screenPosition = position + offset
	    let offset2 = if screenPosition >= rows2
	        then offset - (screenPosition - rows2 + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    let fromages2 = drop (0 - offset2) $ zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese (line == position))
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    ev <- Vty.next_event vt
	    case ev of
	        Vty.EvKey (Vty.KASCII c) [] -> handleKeyboard c position offset2 vt
	        _ -> return vt

Unlike in *NCurses*, no blanking is required.

## Colors
Usage of colors in *Vty* is fairly easy; no need to define color pairs or anything like that. You can use the *Graphics.Vty.Attributes* module in order to change character style. Since all submodules seem to get reimported into *Graphics.Vty*, we can just use that. According to the authors of *Vty*: "Seriously, terminal color support is INSANE."

We can control colors by changing the *Attr* value being passed to *string*. Nicely enough, no padding is required! We can change the colors by using *with_fore_color* and *with_back_color*:
	fromageImage cheese cursor = do
	-- prints out info on a cheese
	    let wfc = Vty.with_fore_color
	    let wbc = Vty.with_back_color
	    let (indicator, useColor) = if cursor
	        then (" > ", True)
	        else ("   ", False)
	    let attr = if useColor
	        then Vty.current_attr `wfc` Vty.black `wbc` Vty.white
	        else Vty.current_attr `wfc` Vty.white `wbc` Vty.black
	    Vty.string attr $ indicator ++ (getName cheese)



This is what the code should look like now:
	#!/usr/bin/env runhaskell
	
	module Main where
	import qualified Graphics.Vty as Vty
	
	fromages =
	    [ "Abondance de Savoie - 1990"
	    , "Banon - 2003"
	    , "Beaufort - 1968"
	    , "Bleu d'Auvergne - 1975"
	    , "Bleu de Gex-Haut-Jura - 1977"
	    , "Bleu des Causses - 1979"
	    , "Bleu du Vercors-Sassenage - 1998"
	    , "Brie de Meaux - 1980"
	    , "Brie de Melun - 1990"
	    , "Brocciu - 1983 (sheep and goats' milk)"
	    , "Camembert de Normandie - 1983"
	    , "Cantal - 1956"
	    , "Chabichou du Poitou - 1990"
	    , "Chaource - 1970"
	    , "Chevrotin - 2002"
	    , "Comté - 1952"
	    , "Crottin de Chavignol - 1976"
	    , "Époisses - 2004"
	    , "Fourme d'Ambert - 1972"
	    , "Fourme de Montbrison - 1972"
	    , "Gruyère - 2007"
	    , "Laguiole - 1961"
	    , "Langres - 1991"
	    , "Livarot - 1972"
	    , "Mâconnais - 2005"
	    , "Maroilles - 1976"
	    , "Morbier - 2000"
	    , "Munster-Géromé - 1969"
	    , "Neufchâtel - 1969"
	    , "Ossau-Iraty - 1980"
	    , "Pélardon des Cevennes - 2000"
	    , "Picodon - 1983"
	    , "Pont l'Evêque - 1976"
	    , "Pouligny-Saint-Pierre - 1972"
	    , "Reblochon - 1958"
	    , "Rigotte de Condrieu - 2008"
	    , "Rocamadour see also Cabecou - 1996"
	    , "Roquefort - 1925"
	    , "Saint-Nectaire - 1955"
	    , "Sainte-Maure de Touraine - 1990"
	    , "Salers - 1979"
	    , "Selles-sur-Cher - 1975"
	    , "Tome des Bauges - 2002"
	    , "Vacherin Mont d'Or - 1981"
	    , "Valençay - 1998"
	    ]
	
	
	getName cheese = cheese
	-- returns the name of a cheese.
	-- This will become more complicated some day.
	
	fromageImage cheese cursor = do
	-- prints out info on a cheese
	    let wfc = Vty.with_fore_color
	    let wbc = Vty.with_back_color
	    let (indicator, useColor) = if cursor
	        then (" > ", True)
	        else ("   ", False)
	    let attr = if useColor
	        then Vty.current_attr `wfc` Vty.black `wbc` Vty.white
	        else Vty.current_attr `wfc` Vty.white `wbc` Vty.black
	    Vty.string attr $ indicator ++ (getName cheese)
	
	allocate = do
	-- sets up Vty
	    vt <- Vty.mkVty
	    return vt
	
	deallocate vt =
	-- frees Vty resouces
	    Vty.shutdown vt
	
	handleKeyboard c position offset vt = case c of
	-- handles keyboard input
	    'q' -> return vt
	    'j' -> work (position + 1) offset vt
	    'k' -> work (position - 1) offset vt
	    _ -> work position offset vt
	
	work requestedPosition offset vt = do
	-- displays cheeses
	    let position = max 0 (min requestedPosition (length fromages - 1))
	    Vty.DisplayRegion cols rows <- (Vty.terminal_handle >>= Vty.display_bounds)
	    let (cols2, rows2) = (fromEnum cols, fromEnum rows)
	    let screenPosition = position + offset
	    let offset2 = if screenPosition >= rows2
	        then offset - (screenPosition - rows2 + 1)
	        else if screenPosition < 0
	            then offset - screenPosition
	            else offset
	    let fromages2 = drop (0 - offset2) $ zip [0..] fromages
	    let fromagesImages = map
	            (\(line, cheese) -> fromageImage cheese (line == position))
	            fromages2
	    let imagesUnified = Vty.vert_cat fromagesImages
	    let pic = Vty.pic_for_image $ imagesUnified
	    Vty.update vt pic
	    ev <- Vty.next_event vt
	    case ev of
	        Vty.EvKey (Vty.KASCII c) [] -> handleKeyboard c position offset2 vt
	        _ -> return vt
	
	main = allocate >>= (work 0 0) >>= deallocate
	-- the main program

## Caveat emptor
This will be the only module where I only have *superlatives* to put in here. The *Vty* module was a breeze to use. My problem was that I couldn't figure out its drawing functions — that is, anything that has to do with the *Image* or *Picture* types. Just didn't seem to work for me, they were eating output or just not doing anything. However, after a quick (and *immediate*!) email exchange with Corey O'Connor, I have figured out what to do. I cannot stress enough how important it is to have such great support from the authors! Corey was really helpful and was invaluable in making this blog post into something which can be recommended to *Vty* users.

## In the end...
I think that both *Vty* is pretty cool. *NCurses* is next up, but it can't match *Vty* in terms of usability. Turns out *HSCurses* is unusable unless the multibyte character problems are fixed.

It might be a *curses* limitation, but it's not acceptable to have the application crashing when output is being done outside of the terminal. A safety net should be in place for this situation. In my experience *Vty* performed best there, simply stopping the cursor at the border.

Each of the frameworks had its own issues:

- *Vty* is missing a simple walkthrough; hopefully this blog post will suffice.
- *HSCurses* doesn't feel too Haskellish, has problems with printing characters, and likes to crash; additionally it doesn't export some useful functions from *curses*
- *NCurses* feels like they promise more than they actually seem to do
- *NCurses* only works if the program is compiled
- all thee frameworks are missing a load of documentation; most importantly ready examples that include every function you might want to use. A function without usage examples is useless, especially given the vastly different APIs that all three try to implement for their more advanced features.

All three are cool; *HSCurses* had issues that would prevent me from using them in release programs (the crashiness). Work on that guys, and you're golden. Let's have a new Vim clone in Haskell, or something.

I am perfectly aware that the code I am showing here might not be the best; maybe I'm missing some important points as to how things should be done; feel free to comment and show how you would do things! It would be cool to see how you can better use *NCurses* and *Vty* in a more functional way rather than having *do* blocks everywhere like in my code.

# ⁂
I think *curses* and similar toolkits are pretty cool; there's a lot to be said about complicated interfaces in text-mode terminals. Sure, you can make the next roguelike with those, but you can also use those libraries to facilitate your work, to make interesting, modal, multi-view user interfaces, and to create a user interface that is different from the command line and geared specifically for the needs of a niche.

</markdown>
