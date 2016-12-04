---
author: Andrew
comments: true
date: 2015-04-14 06:16:13+00:00
layout: post
link: http://ajmccluskey.com/2015/04/clojure-to-haskell-composition/
slug: clojure-to-haskell-composition
title: 'Clojure to Haskell: Composition'
wordpress_id: 624
categories: Programming
tags: clojure functional programming haskell
---

I've been playing with Clojure on and off for the past couple of years and have recently started to learn a little Haskell. I've read through a fair chunk of [Learn you a Haskell](http://learnyouahaskell.com/) and now I'm working through the [CIS194 materials](http://www.seas.upenn.edu/~cis194/lectures.html) from UPenn after being introduced to them at the [Brisbane Functional Programming Group](http://www.meetup.com/Brisbane-Functional-Programming-Group/). Side note: they've been doing their own presentations of the lectures and the videos are on [their Vimeo channel](https://vimeo.com/channels/bfg) (search for "Yorgey").

While tackling the second week's exercises I thought I could write a clean solution using point free syntax based on my experience with composition and partials in Clojure. I couldn't. Unsurprisingly, Clojure's lack of static typing gives it more room to move in these cases.

<!-- more -->

The problem I was trying to solve is counting the number of exact matches between a secret and a guess in the game of [Mastermind](https://en.wikipedia.org/wiki/Mastermind_(board_game)). This is simply the number of corresponding items in two lists that are equal. To give you an idea, here are some types and expected outputs in Haskell.

```haskell
-- BEGIN Excerpt from HW02.hs
-- A peg can be one of six colors
data Peg = Red | Green | Blue | Yellow | Orange | Purple
         deriving (Show, Eq, Ord)

-- A code is defined to simply be a list of Pegs
type Code = [Peg]

exactMatches :: Code -> Code -> Int
-- END DEFINITIONS

-- In ghci
HW02.hs> exactMatches [Red, Green, Blue] [Red, Yellow, Purple]
1
HW02.hs> exactMatches [Red, Green, Blue, Yellow] [Red, Green, Yellow, Blue]
2 
```

In Clojure one could solve this in something roughly equivalent to point free style pretty easily - that is, we could create a function by composing other functions rather than explicitly mentioning the arguments to the function.

```clojure
(def exact-matches
  (comp count (partial filter identity) (partial map =)))

(exact-matches [1 2 1 2] [1 1 1 1]) ;; => 2
```

Now, let's try and define the equivalent in Haskell using ghci.

```bash
Prelude> let exactMatches = length . (filter id) . (zipWith (==))

<interactive>:12:45:
    Couldn't match type `[b0] -> [Bool]' with `[Bool]'
    Expected type: [b0] -> [Bool]
      Actual type: [b0] -> [b0] -> [Bool]
    In the return type of a call of `zipWith'
    Probable cause: `zipWith' is applied to too few arguments
    In the second argument of `(.)', namely `(zipWith (==))'
    In the second argument of `(.)', namely
      `(filter id) . (zipWith (==))'
```

No dice. Turns out Haskell's static typing won't let us do this. Why? Let's look at some types.
    
```bash
Prelude> :t (.)
(.) :: (b -> c) -> (a -> b) -> a -> c
Prelude> :t length
length :: [a] -> Int
Prelude> :t filter id
filter id :: [Bool] -> [Bool]
Prelude> :t zipWith (==)
zipWith (==) :: Eq b => [b] -> [b] -> [Bool]
Prelude> :t length . filter id
length . filter id :: [Bool] -> Int
Prelude> :t filter id . zipWith (==)

<interactive>:1:13:
    Couldn't match type '[b0] -> [Bool]' with '[Bool]'
    Expected type: [b0] -> [Bool]
      Actual type: [b0] -> [b0] -> [Bool]
    In the return type of a call of 'zipWith'
    Probable cause: 'zipWith' is applied to too few arguments
    In the second argument of '(.)', namely 'zipWith (==)'
    In the expression: filter id . zipWith (==)
```


From the above we see that we can compose `length` and `filter id`. This is because the type of `filter id` is `[Bool] -> [Bool]`, the type of `length` is `[a] -> Int`, and the type of `(.)` (the compose operator) is `(b -> c) -> (a -> b) -> a -> c`. If we substitute the types from `length` and `filter id` into compose's type we get:

```haskell
b :: [Bool] -- length's argument type is polymorphic,
            -- but becomes [Bool] in this case
            -- because of (filter id)'s type
c :: Int    -- length's return type
a :: [Bool] -- (filter id)'s input type

([Bool] -> Int) -> ([Bool] -> [Bool]) -> [Bool] -> Int
```


However, when we try to compose `filter id` and `zipWith (==)` the type's don't match. We saw above that, `zipWith (==)` is a function that takes a list of any type that can be tested for equality, and returns a function that takes another list of the same type and returns a list of `Bool`. This may seem strange, but it's how currying works. As a result, the return type of `zipWith (==)` is not equal to the type of the argument in `filter id`; that is, `[b] -> [Bool]` isn't equal to `[Bool]`. Because the types don't match, we can't compose these functions.

But what about Clojure?! Clojure is a dynamic language. If you look at the [source for `partial`](https://github.com/clojure/clojure/blob/599affa78f963da4ea9ea210a76b4b11a35ff9c4/src/clj/clojure/core.clj#L2481) you can see that in this particular case it just returns a function that calls `map` with `=` as its first argument, then passes any additional arguments in. If we pass in two lists, say `[1 2 1 2] [1 1 1 1]`, we end up with an expression that looks like `(map = [1 2 1 2] [1 1 1 1])`. When this expression is evaluated at runtime, it returns a list of `java.lang.Boolean`, which can certainly be passed to `(partial filter identity)`.

None of this is particularly surprising when you think about it, but it was a little jarring coming from Clojure Land where one can compose things on faith. While some might be tempted to hold things like this up as evidence that dynamic typing is better than static, it's important to look at the bigger picture and understand that static typing might be stricter, but it's also more protective. For example, Clojure will happily create functions through composition that cannot possibly be evaluated without error: `(comp count count)` compiles fine, but blows up no matter which arguments you give it. Horses for courses.
