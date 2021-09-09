---
title: 'Haskell - When Are Contravarient Functors Actually Useful?'
description: 
featured_image: '/images/haskell---when-are-contravarient-functors-actually-useful/featured-image.jpeg'
---

There are several resources online that clearly describe Contravariant Functors
and Profunctors. If you have enough background in Functors in Haskell already,
then these resources are pretty straightforward to follow. However, if you are
anything like me, after reading these resources, they left you thinking
thinking _"Great, I know what a Contravariant Functor is, but when and why
should I use them?"_. In this article, I want to give an example of a concrete
situation when you might use them to great effect.

## Rewind to Functors, Applicatives and Monads

When learning Haskell, one of the biggest beginner challenges is getting to
grips with Functors, Applicatives and Monads. However, once you’ve grokked
them, they become second nature, and you find yourself using them extensively.
You don’t actively search for ways to use these abstractions, but rather, your
familiarity with them gives an understanding of when they apply (this is the
same with software design patterns). There are three reasons why a Haskell
programmer will naturally become very familiar with Functors, Applicatives and
Monads:

1. The standard ones (`Maybe`, `Either`, `Reader`, etc.) are incredibly useful.
2. `IO` is a Monad, so you are forced to learn Monads (and, in the process, get
   introduced to the standard set of them).
3. There is special do notation which will confuse you if you don’t understand
   what it is doing.

Once we know about these standard functors, we start to use them everywhere
(they really are useful). As we write functions using them, we recognise places
where we can generalise these functions to the type classes (`Functor`,
`Applicative` or `Monad`).

For the sake of this article, let us simplify the definition of a functor to:

> A box that contains a value of type `a`;
> you cannot access that value directly, but you can _lift_ a function into box
> to apply it to the value using `fmap`.

This is not an accurate definition of a functor, but it is a modeL I think is
helpful for this article.

```haskell
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

## Briefly Introducing the Contravariant Functor

Now let us look at Contravariant Functors.
I'm only going to provide enough context here to continue with this discussion.
For more in-depth introductions, I recommend starting with one of these great resources:

- [Understanding contravariance](https://typeclasses.com/contravariance) & [Understanding profuctors](https://typeclasses.com/profunctors)
- [Contravariant Functors](https://blog.ploeh.dk/2021/09/02/contravariant-functors/)
- [Julie Moronuki - A Gentle Introduction to Profunctors](https://www.youtube.com/watch?v=tfQdtPbYhV0)
- [Phil Freeman - Fun With Profunctors](https://www.youtube.com/watch?v=OJtGECfksds)

Contravariant functors have the arrow of the in the function _lifted_ into the functor reversed.

```haskell
class Contravariant f where
    contramap :: (b -> a) -> f a -> f b
```

This seems impossible at first - if our `f a` is a box containing a value of
type `a`, and we want an `f b`, how are we able to get that with only a
function of `b -> a` availabie?  If we think of our boxes as only containing
a scalar value, then this is impossible; instead, we then our box to contain a
function.

```haskell
newtype Box = Box { unbox :: Input -> Output }
```

Now depending on whether we parameterise the type of `Input` or `Output`, we
can create either a _covariant functor_ or a _contravariant functor_.

**Covariant functor example:**

```haskell
newtype Box a = Box { unBox :: Input -> a }

instance Functor Box where
    fmap :: (a -> b) -> Box a -> Box b
    fmap f (Box g) = f . g
```

**Contravariant functor example:**

```haskell
newtype Box a = Box { unBox :: a -> Output }

instance Contravariant f where
    contramap :: (b -> a) -> Box a -> Box b
    contramap f (Box g) = g . f
```

For the rest of this article I want to redefine my earlier definition of
covariant functors to:

> A box which contains a **process that produces a value** of type `a`.
> You cannot access that process directly, but you can _lift_ a function into 
> the box to apply to the **result** of that process using `fmap`.

Let's also define contravariant functors as:

> A box which contains a process that **receives a value** of type `a`.
> You cannot access that process directly, but you can _lift_ a function into
> the box to apply to the **input** before the process using `contramap`.

## Using Specific Types

Everything I've talked about so far is about what a contravariant functor is,
but I've still not address why they are useful. Before I can talk about why
contravariant functors are useful, there's another thing we need to discuss -
wrapping types.

Something that we like to do in Haskell is wrapping primitive types into custom
types.  An example would be that rather than passing a `String` to a function,
we wrap `String` in a `Username` type to signify that it is something more
specific than a generic string.

```haskell
newtype EmailAddress = EmailAddress String

sendWelcomeMessage :: EmailAddress -> IO ()
```

We get a few benefits from this:

1. Make intent clear to the reader.
2. The typechecker ensures that we do not accidentally pass the wrong argument at compile time.
3. Using _smart constructors_, we can ensure that invalid values are impossible to create.

### Encoder/Decoder Example

Now let us consider that a function is also a primative value; we often want to
pass a function as an argument to another function. As an example, we will
create a pair of functions that saves/loads string value to/from a file.

```haskell
saveStringToFile :: FilePath -> String -> IO ()

loadStringFromFile :: FilePath -> IO String
```

Let us extend these to also have an encode/decode function passed in.  The
encode function takes the unencoded string and returns the encoded string; the
decode function does the opposite. The save function should take an encoder,
and the load function should take a decoder.

```haskell
saveStringToFile :: (String -> String) -> FilePath -> String -> IO ()

loadStringFromFile :: (String -> String) -> FilePath -> IO String
```

The types here are not very clear, we could try adding type aliases to make it
more obvious:

```haskell
type Encoder = String -> String
type Decoder = String -> String

saveStringToFile :: Encoder -> FilePath -> String -> IO ()

loadStringFromFile :: Decoder -> FilePath -> IO String
```

This makes things clearer to the reader, but it provides no extra type-safety;
there is nothing stopping someone from passing a decode function to the save
function. Another option would be to create `EnecodedString` and `PlainString`
types:


```haskell
newtype PlainString = PlainString String
newtype EncodedString = EncodedString String

saveStringToFile :: (PlainString -> EncodedString) -> FilePath -> String -> IO ()

loadStringFromFile :: (EncodedString -> PlainString) -> FilePath -> IO String
```

This works. It might be exactly what your program needs; not accidentally using
an unencoded string when you should be using an encoded one, might be something
you want the compiler to stop you from doing). However,  for this example, let
us consider that this is unnecessary overhead.  Instead, let us do what we do
with other primitive values and wrap the functions in a `newtype`:

```haskell
newtype Encoder = Encoder { encode :: String -> String }
newtype Decoder = Decoder { decode :: String -> String }

saveStringToFile :: Encoder -> FilePath -> String -> IO ()

loadStringFromFile :: Decoder -> FilePath -> IO String
```

We are using our type to specify that `Encoder` and `Decoder` are each a
specific subsets of `String -> String`. Now we can defined some encode and
decode functions:

```haskell
rot13Encoder :: Encoder

rot13Decoder :: Decoder
```

And we could use them like so:

```haskell
main :: IO ()
main = do
    saveStringToFile rot13Encoder "encoded.txt" "Hello File!"
    decodedString <- loadStringFromFile rot13Decoder "encoded.txt"
    print decodedString
```

### Generalising the Example

Let us now consider another set of load and save functions that work with
`Int`, but still persist the value as an encoded string in the file. First, we
will try without our `Encoder` and `Decoder` types:

```haskell
saveToStringFile :: (String -> String) -> FilePath -> String -> IO ()

loadStringFromFile :: (String -> String) -> FilePath -> IO String

saveToIntFile :: (Int -> String) -> FilePath -> Int -> IO ()

loadIntFromFile :: (String -> Int) -> FilePath -> IO Int
```

We can keep using the rot13 encoder and decoder:

```haskell
rot13Encoder :: String -> String

rot13Decoder :: String -> String
```

And let's serialise numbers as Roman Numerals:

```haskell
intToRoman :: Int -> String

romanToInt :: String -> Int
```

We can now use function composition to easily convert to Roman Numerals before
encoding and from Roman Numerals after decoding.

```haskell
main :: IO ()
main = do
    saveStringToFile (rot13Encoder . intToRoman) "encoded.txt" "Hello File!"
    decodedString <- loadStringFromFile (romanToInt . rot13Decoder) "encoded.txt"
    print decodedString
```

This works out nicely; we can compose our `rot13Encoder` and `rot13Decoder`
with other functions to work with any type! Now, what happens when we
reintroduce our `Encoder` and `Decoder` types (note we now have to parameterise
them by type):

```haskell
newtype Encoder a = Encoder { encode :: a -> String }
newtype Decoder a = Decoder { decode :: String -> a }

rot13Encoder :: Encoder String

rot13Decoder :: Decoder String

intToRoman :: Int -> String

romanToInt :: String -> Int

saveStringToFile :: Encoder String -> FilePath -> String -> IO ()

loadStringFromFile :: Decoder String -> FilePath -> IO String

saveToIntFile :: Encoder Int -> FilePath -> Int -> IO ()

loadIntFromFile :: Decoder Int -> FilePath -> IO Int
```

This all looks good, but how are we going to compose the `rot13` functions with the `roman` ones?

## Using Contravariant Functors

The answer is to use covariant and contravariant fuctors.
If we define our `Encoder` as a contravarient functor and our `Decoder` as a covariant functor, then we get what we need:

```haskell
newtype Encoder a = Encoder { encode :: a -> String }

instance Contravariant Encoder where
    contramap :: (b -> a) -> Encoder a -> Encoder b
    contramap f (Encoder encode) = encode . f

newtype Decoder a = Decoder { decode :: String -> a }

instance Functor Decoder where
    fmap :: (a -> b) -> Decoder a -> Decoder b
    fmap f (Decoder decode) = f . decode
```


We can now use `fmap` and `contramap` to compose the `rot13` functions with the `roman` ones:

```haskell
main :: IO ()
main = do
    saveStringToFile (fmap rot13Encoder intToRoman) "encoded.txt" "Hello File!"
    decodedString <- loadStringFromFile (contramap rot13Decoder romanToInt) "encoded.txt"
    print decodedString
```

And of course, we can use fancy operators to look clever if we wanted:

```haskell
main :: IO ()
main = do
    saveStringToFile (rot13Encoder <$> intToRoman) "encoded.txt" "Hello File!"
    decodedString <- loadStringFromFile (romanToInt >$$< rot13Decoder) "encoded.txt"
    print decodedString
```

### WHY?!

First of all, using specific types tells both the reader and the compiler what
type of values are expected. For the reader, this makes the intend to the
program clearer, and for the compiler, it stops us passing incorrect values.

Secondly, once we recognise these abstractions and start to incoperate them
into out types, then we will start to see more general concepts appearing in
our code (in the same way that use use `Functor` and `Moand` extensively).

## Profunctors

A quick note on profuctors, they are also a box with a process in it, but the 
process is parameterised in both input and output types. If are passing a
wrapped function where you want to be able to compose both before and after then
you can use a profuctor. For example, the following would be a good opportunity
to use a Profuctor:

```haskell
newtype DataProcessor a b = DataProcessor { process :: a -> b }

sendData :: DataProcessor String String -> Source -> Destination -> IO ()
```

### The Category Typeclass

When using Profunctors we can also consider implementing `Category` which will
enable use to compose the processes using `>>>`.

```haskell
instance Category DataProcessor where
  id dataProcessor                      = dataProcessor
  (DataProcessor f) . (DataProcessor g) = DataProcessor (f . g)

processor1 :: DataProcessor String Int

processor2 :: DataProcessor Int String

main :: IO ()
   source <- getSource
   destination <- getDestination
   sendData (processor1 >>> processor2) source destination

```

## Summary

If any category theory people are reading this article then they are probably
screaming at me. I'm ultimately saying ignore the theory and utilise the
typeclasses (although I'd still say to try and conform to the laws
abstractions). I'm not saying this because it is the right thing to do, but
rather, in the hope that by starting this way it provides as a gateway to
gaining some intuition about these concepts.

When passing passing functions as argument, ask if all functions of the given
type are valid. If all functions are not valid, consider creating a `newtype`
for the subset of functions when are valid. Use _covariant functors_,
_contravariant functors_ and _profunctors_ to make them composable.
The see what other benefits fall out of using these abstractions.

