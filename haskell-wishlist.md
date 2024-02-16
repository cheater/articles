Title: My Haskell Wishlist
slug: haskell-wishlist


I really like Haskell and there are a few things that I would love to see in it from a practical standpoint.


# Automatic value inference

`auto`: a keyword in Haskell which is kind of like a hole or like `undefined`, in that it can assume any type in any place; when it is seen by the compiler, the compiler tries to infer its type, and then searches for the unique value in the current name space that fits that type. If such a type is not found, a compiler error occurs. See below under "Interactive development" to see more.

You could probably give type hints about the auto value, e.g. `map (auto :: _ -> Bool) myList`

You could probably also have some smart sort of "auto via" like "deriving via", but I don't know what the best design for that would be.

In addition to free theorems, this could be a very useful tool for Haskell development.


# Interactive development

Interactive compiler suggestions: often, GHC will suggest things like adding an extension to a file or a package to imports or a module to exports. Allow the user to say yes or no to those. In that case, the compiler automatically writes the change to the file. Make interactive mode the default mode for when running `cabal build|run|test|etc` in a TTY. Create a new compiler flag called noauto that fails on encountering auto. (As an aside, there should also be a flag like that for undefined, for holes, etc).

If an interactive compiler suggestion session finds the "auto" keyword and has several options for it, then it should show a TUI list where you can use up/down arrows or j/k to go through the possible options, and by using enter you select the function to be patched into that spot ("auto" is replaced with the new function). This should show 10 items at a time with pagination, and if there are more than 50, it should just error out. This last setting (50 or whatever) should be settable using a config file.

ghci should have a `:doc` command that displays the doc for a function. example: `:doc Data.List.reverse`. Functions that have been applied to something don't need to show a doc anymore, only a function that hasn't been applied needs to carry the doc. Similarly, a `:src` function should be available to show the source for a function. Preferably, the `:doc` is opened in vim with similar facilities as the vim `:help` function, with `ctrl-]` etc.

ghci should allow writing out some of the functions you have defined in the repl to a file by doing `:save`. Example: `:save src/Foo.hs` will show you a list of possible things to write to the file, and you can check the box next to them using an interface that allows using the up and down arrows and j/k to move, space and enter to select/unselect a thing, shift up/down and shift j/k to select multiple things ("drag" them), A to toggle all on/off/undo. A-undo works in the following way: if you have options 1 2 and 3 selected but 4 and 5 are unselected, A will select all, then another A will select none, then another A will return to 1 2 and 3. If after the first or second A you make a change (select or deselect something), then the state machine gets reset and now pressing A again will select all, then select none, then undo to the new state. If you don't want interactivity, use `:saveall`. You should also be able to use `:load` on those files. This can easily create an entry point to something like an interactive notebook in ghci. Eventually, past entries could be made editable in ghci, to make it even more notebook-like. This should be doable in the terminal using a TUI, rather than *only* in the browser.


# Build

Have the ability to create a Software Bill Of Materials (SBOM) for a cabal package.

Ability to preclude a certain package version range from being used, even in transitive dependencies.

Ability to have build targets, eg "dev" vs "debug" vs "release". For example, you might want to warn on `undefined` in dev, but have a compilation error on debug or release. You might want to have debug symbols in dev and debug, but not in release. Etc.

Version pinning using cryptographically strong hashes of packages.

Signed packages.

Vendoring.

None of these really need an explanation - google can explain the necessity for those better than I can make this happen within the confines of this document.


# State machines

First-class state machines: a lot of things in programming are state machines. make it possible to express them using first-class syntax that is dedicated to state machines in the clearest possible way. a single value with a type that is a state machine is called a state. make it possible to match against state using case.

State machines are a specific case that is a little more than an Enum but a lot less than an ADT or GADT. There is 

Example state machines:

```
state FlipFlop where
  turnOff :: On -> Off
  turnOn :: Off -> On
```

Note that this tells us that during a single transition, if we *know* a transition happens, the FlipFlop can only transition to the opposite state and cannot transition eg from On to On.

```
state CSV where -- simple CSV parser
  Normal -- you can start in Normal state
  Normal -> Quote -- we were parsing a field and we encountered a "double quote", meaning that until it is matched we will keep parsing everything as one field
  Quote -> Normal -- we were in quote mode and we encountered another quote character, return to normal mode
  Normal -> Separator -- we were parsing a field and we encountered a comma that separates fields. We store away the previous field and start on a new one
  Separator -> Normal -- after a comma, we resume normal state
```

Note that by looking at the above code we can immediately tell that if we're in the Quote mode we cannot start a new field until we encounter a new quote character.

```
state HTTP -- a state machine for HTTP requests. It would be very
           -- complicated, and yet I bet some of you would be able
           -- to write it in a way that is very readable using the
           -- syntax above.
  ...
```

The problem with state machines as they are being done currently in Haskell (and in almost every other programming language) is that they are obscured by their encoding into whatever code tries to represent them. For example, you can have state machines loosely represented by GADTs, but that representation is exactly that - loose. When looking at a GADT value you can't really tell if the value is being used as a state machine or not; often, GADT values can be instantiated without prerequisites, whereas in state machines we often have clearly defined initial states. With GADTs, you can transition from one constructor to another, without a care in the world; in state machines, you want rigidly defined transitions.


Ensuring state machine transitions:

Create a new type constructor called `StateTransition` that takes the name of a state machine and ensures that a correct transition occurs.

For example:

```
parseCsv_ :: Char -> StateTransition CSV
```

this ensures that `parseCsv_`, which is fed a character at a time, transitions correctly.

Create a new arrow (`~>` or whatever) that translates to `StateTransition`:

```
parseCsv_ :: Char -> CSV ~> CSV

parseCsv_ c Quote = Separator -- compilation error

parseCsv_ c Quote = auto -- see "Auto inference" above
```


Note that `parseCsv_` doesn't return a value, meaning that while it changes the indicated state in the `CSV` state machine, it doesn't actually do anything useful at all. I don't really know of an elegant way to make a function signature both denote a state transition but also return a value (like any productive function would), so that is a thing that needs improvements. Maybe "Multiple type systems" below is an answer.

The importance of state machines cannot be understated. I've spent the last several years doing audits on a lot of sensitive code in various languages and inevitably, bad encodings of state machines return over and over with serious bugs or even exploits, some of which have real impact on people's lives and personal security.


# Memory layout and reasoning about memory

I would like to be able to indicate and rason about the memory layout of values of a specific type. In fact, I would like to be able to create something that, in memory, exactly looks like a C struct - for any ADT value. This is necessary for many optimizations that are otherwise not possible. I would also like to be able to reason about whether specific values are on the same memory page and indicate that they should be. Finally, I would like to be able to run C FFI on such struct-like memory values directly, while memory access is frozen to Haskell (eg in a multithreaded scenario).

I would like to be able to reason about the amount of memory copies a certain piece of code does. If I have a large value, or a lot of small values, and they are being needlessly copied, I would like to know about this and I would like to be able to prevent this.


# Multiple type systems

Being able to reason using the Haskell type system is great. But it is not the only kind of judgment we should be able to make about our code. There are other types of judgments we would like to make:


## 1. What is the time complexity of the function?

For example, take this:

```
sort :: [a] -> [a] -- doesn't tell us much about performance
sort xs = ...
```

```
sort :: [a] -> [a]
sort :: n -> LessEqO (n * Log n) -- tells us what we need to know: time complexity is at most O(n log n)
sort xs = ...
```

note that in the above code, the two type signatures are present at the same time, and they do not interfere with each other.

Given useful primitives, ime complexity can be inferred about a lot of day to day code. Where it can't be inferred, a type annotation can be done. Where it can be inferred, eg in `sort` above, we could say that the code infers to being `sort :: n -> O (n * n)` but it is specified to being `LessEqO (n * Log n)`, so compilation fails, as desired.


## 2. What is the space complexity of the function?

Similar as the above, with the addition that you should be able to both define the desired time and space complexity of a function at the same time.


## 3. State transitions performed in a function. Example:

```
parseCsv :: CSV -> Char -> FieldAccum -> FieldAccum
parseCsv csvState char acc = case csvState of
  Normal -> ...
  Quote -> ...
  ...
```

The above does not ensure that we never go from Quote to Separator. However, we can add this:

```
parseCsv :: CSV ~> CSV
parseCsv :: CSV -> Char -> FieldAccum -> FieldAccum
parseCsv csvState char acc = ...
```

Perhaps one way to make those two signature worlds work together would be to treat every signature that isn't the usual Haskell type signature in a similar way one would treat a monad, in that anywhere in pure code you can "reach out to" the other signature world and inform what is happening in that world, eg using a new keyword called `yield`. For example:

```
parseCsv :: CSV -> _ -> _ ~> CSV
parseCsv :: CSV -> Char -> FieldAccum -> FieldAccum
parseCsv csvState char (CurrentField xs) = case csvState of
  Normal -> case char of
    '"' -> do
      yield Quote -- correct transition from 
      acc
    ',' -> do
      yield Separator
      FieldFinished xs
    c -> do
      yield Normal
      CurrentField (xs ++ [c])
      
  Quote -> case char of
    '"' -> do
      yield Normal
      acc
    ',' -> do
      yield Separator -- compiler error
      FieldFinished xs
```

## Other examples

Other examples of useful judgments about values include:

4. Does the code ever touch the GPU?

5. Does the code ever use a specific kind of cryptography?

6. "This value and any value based off of it needs to be deleted after at most x seconds" - e.g. when processing a password, to prevent memory-dumping attacks; or when creating a streaming proxy, in order to make sure that dead cache does not remain in memory as a leak.

7. "This is the layout of this value"


## Discussion

In all these examples (1-3), the additional signatures are optional and only serve to further restrict the kinds of code that we allow to compile. If someone doesn't care about state transitions or about space performance, they don't need to specify this; the compiler is free to infer these signatures, but there is nothing stopping it from accepting code based on those inferred signatures. As with any inference, even if you don't specify a restricting signature for some value, inferring its signature is still useful, because a value composed out of that value might have a typed-out signature.

It might, technically, be possible to express these as monads, but there would be no elegant, easily readable syntax for state machines, and there isn't a good way to make specification of such things optional. So eg to have time complexity inferrable on code built up of Prelude functions, we would need to pretty much have all of Haskell live in a monad, which would be uncouth.


# More, easier FFIs

I would like to be able to easily import modules and run functions from the following languages: JavaScript, Python, Rust. While this is not always going to be typesafe, I would like to be able to make that trade off for keeping a mostly-Haskell codebase that reaches out to libraries for functionality that is not yet present in the Haskell ecosystem. It is clear that even if most of them are weird garbage, one million npm packages are still going to provide a lot of easy to use value that simply isn't available in Haskell.

This is the ease with which I would like to be able to use JS functions:

```
module Main where

importjs "left-pad" (leftPad)

main = putStrLn $ leftPad "Hello, World!" 5
```

Yes, there are some things GHC won't be able to infer. Yes, they will be runtime errors. Yes, I am fine with that.

While left-pad is a simple example, being able to use more advanced things is really useful. You can't tell me that e.g. being able to do this with scipy or numpy isn't a good idea.

As code like that is written, people will be able to replace portions of it with higher-quality Haskell code in their own time, while continuously developing the same Haskell codebase.

Haskell's abilities as the "polyglot language" (in that it's great for writing compilers, DSLs, parsers, interpreters, and protocols) should also extend to it being able to *speak* other languages in a fluent manner.


# Implicit function definition, logic programming using predicates

An implicit function in mathematics is a function which is defined in the following way:


```
-- let's say f is a function in three-space
P(f, x, y, z) r Q(f, x, y, z)
```

This means that there is some sort of formula P that can involve free variables f, x, y, and z, and another such formula Q, and there is a relation between them called r (for example, the equality relation `=` or inequality `>` or `<` etc or set relations eg `\in`). We assume that the formula `P(f, x, y, z) r Q(f, x, y, z)` defines a unique function f. That is implicit function relation.

Examples:

```
P(f, x, y, z) := f(x, y, z)
r := =
Q(f, x, y, z) := x + y + z
```

then,

```
P(f, x, y, z) r Q(f, x, y, z)
```

translates to

```
f(x, y, z) = x + y + z
```

More elaborate examples can be found here: https://en.wikipedia.org/wiki/Implicit_function_theorem

In Haskell, we are essentially limited to `P := f`. While Q can mention f and all of its arguments (hence we can have recursion, for example `f (x:xs) = (f xs) ++ [x]`), the left side is sadly bereft of any sort of complexity.

It would be nice to be able to define things, eg

```
g :: Bool -> Bool -> Bool -> Bool
g a b c = a && b `xor` c -- boolean operations

f a b = c where
  g a b c == True
```

In the above code, `f a b` evaluates to such a value `c` that `g a b c` evaluates to `True`. It's not hard to infer the value when compiling because the logic here is so simple. It requires a symbolic representation of g, which the compiler already has anyways, so that's not an issue. The type of `f` can easily be inferred from its code.

This can be extended to more logic functions, branches, case statements, and various other code that is in some way finite.

This plays into a larger theme of logic programming in Haskell. Notice that in the above code, one of the lines in a `where` block isn't an assignment, but is instead an expression that can evaluate to a truth value. This is a predicate. We can use predicates like this for a bunch of other stuff, including correctness checking at value level. Such predicates should be checkable at compile time and would greatly improve the safety of Haskell code. (obviously predicates would also work in `let` blocks and in `do`). If a value cannot be inferred (eg due to types being unsuitable for this), the compiler should produce an error.
