---
title: Haskell - Mapping with State
description: Any non-side-effecting thing you would do with a loop can be re-written with a fold, but it's not always pretty.
featured_image: '/images/haskell---mapping-with-state/featured-image.jpg'
---

These days, the functions `map`, `filter` and `fold`/`reduce` are pretty well
known by most programmers. Any pure data manipulation performed in a loop can
be re-written with a `fold`, but it's not always easy to understand. We use
`map` and `filter` when we can to make things clearer, and `fold` is useful
when _folding_ a collection down to a single value. However, for more complex
loops, most would argue that an imperative loop is easier to understand than
trying to wrap up all the state in the accumulator of a _fold_. In this
article, we look at a situation where using `fold` to _map over a collection
while maintaining state_ can be clarified by mapping over a `State` monad.

## The Challenge

The `map` function lets us transform each _element_ in a list, like this:

```haskell
newList = map (+1) [1, 2, 3]
-- newList is [2, 3, 4]
```

But what if we want to do something like this (Javascript):

```javascript
function addAnIncrementingAmount(numbers) {
  let result = [];
  let amountToAdd = 0;

  for (const number of numbers) {
    result.push(number + amountToAdd);
    amountToAdd++;
  }

  return result;
}

const newList = addAnIncrementingAmount([1, 2, 3]);
// newList is [1, 3, 5]

```

Even though the result is a list _mapping_ each element from an element
provided in the argument, we are unable to use `map` because the calculation
for each element is dependant on the previous one. Therefore, we have to use
`foldl` instead:

```haskell
addAnIncrementingAmount :: [Int] -> [Int]
addAnIncrementingAmount numbers =
  fst $ foldl addAmount addAmount ([], 0) numbers

addAmount :: ([Int], Int) -> Int -> ([Int], Int)
addAmount (result, amountToAdd) number =
  (result ++ [number + amountToAdd], amountToAdd + 1)
```

While this code is fairly simple (because the example itself is very simple),
if we had more stateful values used in the calculation then it would get more
complex. Also, the `addAmount` is trying to do two things at once - i.e. build
the resulting list and perform the calculations.

## Generalising with a new higher-order function

Let's start by separating the calculation from the _stateful mapping_. We can do
this by creating a new `mapWithState` higher-order function.

```haskell
addAnIncrementingAmount :: [Int] -> [Int]
addAnIncrementingAmount numbers = mapWithState addAmount 0 numbers

mapWithState :: (a -> state -> (b, state)) -> state -> [a] -> [b]
mapWithState f state xs = fst $ foldl f ([], state) xs

addAmount :: Int -> Int -> (Int, Int)
addAmount number amountToAdd = (number + amountToAdd, amountToAdd + 1)
```

This is a nice separation of concerns. The `mapWithState` function is concerned
with building the resulting list and passing the state through, while the
`addAmount` function is only concerned with our calculation.  This is a good
start, but it's not clear in the type of `addAmount` which argument is the
state and which is the element we are _mapping_ - can we do something about
that?

The type signature should give us a clue as to what we could try. The fact that
it returns a tuple of `(result, newState)` should make us think about the
`State` monad.

## The `State` Monad

`State` is available via `Control.Monad.State` of the `mtl`[^1] package. It is
defined like this:

[^1]: [Hackage documentation for the mtl library](http://hackage.haskell.org/package/mtl)

```haskell
newtype State s a = State { runState :: State s a -> s -> a }
```

And is used like this:

```haskell
import Control.Monad.State (State, get, modify, put, evalState)

performCalculation :: Int -> Int
performCalculation n = evalState statefulCalculation n

statefulCalculation :: State Int Int
statefulCalculation = do
  initalValue <- get
  put (initialValue + 3)
  modify (*2)
  newValue <- get 
  return (newValue - initalValue)
```

Which would be be the equivilent to this in Javascript:

```javascript
function performCalculation(initalValue) {
  let newValue = initalValue + 3
  newValue = newValue * 2
  return (newValue - initalValue)
}
```

While the example above isn't particularly beautiful, I hope it provides enough
information about using the `State` monad for our example. One additional point
about `State` (or any other monad) is that its context is maintained through
functional calls within the monad. Let's see that:

```haskell
import Control.Monad.State (State, get, modify, put, evalState)

performCalculation :: Int -> Int
performCalculation n = evalState statefulCalculation n

statefulCalculation :: State Int Int
statefulCalculation = do
  initalValue <- get
  add 3
  multiply 2
  newValue <- get 
  return (newValue - initalValue)

add :: Int -> State Int ()
add amount = modify (+amount)

multiply :: Int -> State Int ()
multiply amount = modify (*amount)
```

### Back to our example

Now we know about the `State` monad, let's try re-writing the `addAmount`
function to use it:

```haskell
addAmount :: Int -> State Int Int
addAmount number = do
  amountToAdd <- get
  modify (+1)
  return (number + amountToAdd)
```

Now the type tells us something interesting about the function - _it takes an
Int and returns an Int, but it also maintains some state (also of type Int)_.
But how do we use it?

## `Traversable`

We know that we want to our `addAmount` function `map` over our input list, so
what do we get if we try that now?

```
Prelude Control.Monad.State> :t map addAmount [1, 2, 3]
map addAmount [1, 2, 3] :: [State Int Int]
```

We've individually added each element into a `State` and ended up with `[State
Int Int]`, but that's not what we need, we want to compute the entire list
inside `State` - i.e. `State Int [Int]`. Wouldn't it be great if there was a
simple function to convert from what we have into what we want - i.e.  `[State
Int Int] -> State Int [Int]`.

It turns out that lists implement the typeclass `Traversable`. If we look at
the definition of `Traversable` then we see this:

```haskell
class (Functor t, Foldable t) => Traversable (t :: * -> *) where
  traverse :: Applicative f => (a -> f b) -> t a -> f (t b)
  sequenceA :: Applicative f => t (f a) -> f (t a)
  mapM :: Monad m => (a -> m b) -> t a -> m (t b)
  sequence :: Monad m => t (m a) -> m (t a)
```

If we represent our list with `Traversable t` and our `State` monad with `Monad
m`, then the function we after would have the type `t (m a) -> m (t a)` -
that's exactly the type of `sequence` in the typeclass above! Let's try using it:

```haskell
addAnIncrementingAmount :: [Int] -> [Int]
addAnIncrementingAmount numbers =
  evalState (sequence (map addAmount numbers)) 0

addAmount :: Int -> State Int Int
addAmount number amountToAdd number = do
  amountToAdd <- get
  modify (+1)
  return (number + amountToAdd)
```

This works exactly as expected. In fact, this combination of `map` and
`sequence` is so useful, that if we look at the type signatures long enough, we
see that `mapM` in `Traversable` is the composition of these two functions:

```haskell
mapM :: (a -> m b) -> t a -> m (t b)
```

Let's try `mapM`:

```haskell
addAnIncrementingAmount :: [Int] -> [Int]
addAnIncrementingAmount numbers =
  evalState (mapM addAmount numbers) 0

addAmount :: Int -> State Int Int
addAmount number amountToAdd number = do
  amountToAdd <- get
  modify (+1)
  return (number + amountToAdd)
```

At this point, I hope that I've managed to practically show how `State` and
`mapM` can be used together without digging too deeply into the details
(there's plenty of excellent resources on this elsewhere [^2]). However, my
example so far is not that useful, let's look at something a bit more complex.

[^2]: [Haskell Programming from first principles](http://haskellbook.com/) and [Learn You Some Haskell](http://learnyouahaskell.com/) both explain these concepts from the ground up.

## The Bowling Game Kata

The [Bowling Game Kata](https://kata-log.rocks/bowling-game-kata) is a
well-know Test-Driven Development exercise where the goal is to calculate the
score of a game of ten-pin bowling.

A typical Javascript solution might look something like this:

```javascript
class Game {
  constructor() {
    this.rolls = [];
  }

  roll(pins) {
    this.rolls.push(pins);
  }

  score() {
    let score = 0;
    let framePointer = 0;

    for (const frame of Array(10).keys()) {
      if (this.isStrike(framePointer)) {
        score += this.scoreStrike(framePointer);
        framePointer += 1;
      } else if (this.isSpare(framePointer)) {
        score += this.scoreSpare(framePointer);
        framePointer += 2;
      } else {
        score += this.scoreNormalFrame(framePointer);
        framePointer += 2;
      }
    }

    return score;
  }

  isStrike(framePointer) {
    return this.rolls[framePointer] === 10;
  }

  isSpare(framePointer) {
    return this.rolls[framePointer] + this.rolls[framePointer + 1] === 10;
  }

  scoreStrike(framePointer) {
    return 10 + this.rolls[framePointer + 1] + this.rolls[framePointer + 2];
  }

  scoreSpare(framePointer) {
    return 10 + this.rolls[framePointer + 2];
  }

  scoreNormalFrame(framePointer) {
    return this.rolls[framePointer] + this.rolls[framePointer + 1];
  }
}
```

The `score` function has a `for` loop in it so this is the bit which will need
to be re-written to convert it to Haskell. Let's start by extracting the
scoring of each frame and then add up the results at the end.

```javascript
  score() {
    let frameScores = [];
    let framePointer = 0;

    for (const frame of Array(10).keys()) {
      const [frameScore, rollsInFrame] = this.scoreFrame(framePointer);
      frameScores.push(frameScore);
      framePointer += rollsInFrame;
    }

    return frameScores.reduce((score, frameScore) => score + frameScore);
  }

  scoreFrame(framePointer) {
    if (this.isStrike(framePointer)) {
      return [this.scoreStrike(framePointer), 1];
    } else if (this.isSpare(framePointer)) {
      return [this.scoreSpare(framePointer), 2];
    } else {
      return [this.scoreNormalFrame(framePointer), 2];
    }
  }

```

Now we've extract `scoreFrame` we can convert it to Haskell by replacing the
`for` loop with a `foldl` (the implementation below isn't like for like, but
it's close enough):

```haskell
score :: [Int] -> Int
score rolls = sum $ fst $ foldl scoreFrame ([], rolls) [1..10]

scoreFrame :: ([Int], [Int]) -> Int -> ([Int], [Int])
scoreFrame state@(frameScores, rolls) _frame
  | isStrike rolls = scoreStrike state
  | isSpare rolls  = scoreSpare state
  | otherwise      = scoreNormal state

isStrike :: [Int] -> Bool
isStrike rolls = allPins $ scoreRolls 1 rolls

isSpare :: [Int] -> Bool
isSpare rolls = allPins $ scoreRolls 2 rolls

scoreStrike :: ([Int], [Int]) -> ([Int], [Int])
scoreStrike (frameScores, rolls) =
  (frameScores ++ [scoreRolls 3 rolls], moveFrame 1 rolls)

scoreSpare :: ([Int], [Int]) -> ([Int], [Int])
scoreSpare (frameScores, rolls) =
  (frameScores ++ [scoreRolls 3 rolls], moveFrame 2 rolls)

scoreNormal :: ([Int], [Int]) -> ([Int], [Int])
scoreNormal (frameScores, rolls) =
  (frameScores ++ [scoreRolls 2 rolls], moveFrame 2 rolls)

scoreRolls :: Int -> [Int] -> Int
scoreRolls numRolls rolls = sum $ take numRolls rolls

moveFrame :: Int -> [Int] -> [Int]
moveFrame numRolls rolls = drop numRolls rolls

allPins :: Int -> Bool
allPins = (== 10)
```

We can see here that there's a lot of confusing looking type signatures.
Although this could be greatly improved by using explicit types rather than
`Int` everywhere, there is still a lot of noise from having to manually thread
the state through everywhere.

### Refactoring to Use `State`

Now lets take a look at a version which makes use of `State`:

```haskell
import Control.Monad (mapM)
import Control.Monad.State (State, get, put, modify, evalState)
import Control.Conditional (condM, otherwiseM)

score :: [Int] -> Int
score rolls = sum $ evalState (mapM scoreFrame [1..10]) rolls

scoreFrame :: Int -> State [Int] Int
scoreFrame _frame =
  condM [ (isStrike,   scoreStrike)
        , (isSpare,    scoreSpare)
        , (otherwiseM, scoreNormal)
        ]

isStrike :: State [Int] Bool
isStrike = allPins <$> scoreRolls 1

isSpare :: State [Int] Bool
isSpare = allPins <$> scoreRolls 2

scoreStrike :: State [Int] Int
scoreStrike = do
  score <- scoreRolls 3
  moveFrame 1
  return score

scoreSpare :: State [Int] Int
scoreSpare = do
  score <- scoreRolls 3
  moveFrame 2
  return score

scoreNormal :: State [Int] Int
scoreNormal = do
  score <- scoreRolls 2
  moveFrame 2
  return score

scoreRolls :: Int -> State [Int] Int
scoreRolls numRolls = sum . take numRolls <$> get

moveFrame :: Int -> State [Int] ()
moveFrame numRolls = modify (drop numRolls)

allPins :: Int -> Bool
allPins = (== 10)
```

While there are a few bits in this version which require some understanding, it
does remove a lot of the clutter from the previous version. It also makes a
clear separation between the state and the frame number itself (which
incidentally is never actually used).

## Final Worlds

I don't claim that using `mapM` instead of `foldl` is always clearer, and I
don't claim that either of the implementations of the Bowling Game scoring
exercise above is the best way to solve this in Haskell, but I do hope you have
enjoyed this article and perhaps learned something new along the way.

---

Featured photo by [Michelle McEwen](https://unsplash.com/@michellem18) on [Unsplash](https://unsplash.com/).
