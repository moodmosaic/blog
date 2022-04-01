---
layout: basic
title: Generators and the 'choose' function
summary:
id-image: 8
tags:
    - FSharp
    - FsCheck
---

Test data is produced, and distributed, by test data generators.

Both QuickCheck and FsCheck provide default generators for primitive types, but it's also possible to build custom generators for custom types.

Generators have types of the form `Gen a`, where `a` is the type of the test data to be produced.

## QuickCheck

Generators are built from the function [`choose`](https://hackage.haskell.org/package/QuickCheck-2.7.6/docs/src/Test-QuickCheck-Gen.html#choose), which makes a random choice of a value from a range, with a uniform distribution in the closed interval *[a, a]*.

``` haskell
choose :: Random a => (a, a) -> Gen a
```

The type `Gen` is an instance of Haskell's [`Monad`](https://hackage.haskell.org/package/base-4.7.0.2/docs/Control-Monad.html#t:Monad). This involves implementing its minimal complete definition:

``` haskell
return :: a -> Gen a
(>>=) :: Gen a -> (a -> Gen b) -> Gen b
```

### Example: Take a random element from a list

Using *do* notation:

``` haskell
import Test.QuickCheck

takeFromList :: [a] -> Gen a
takeFromList xs =
    do i <- choose (0, length xs - 1)
       return $ xs !! i
```

A possible translation into vanilla monadic code:

``` haskell
import Test.QuickCheck

takeFromList :: [a] -> Gen a
takeFromList xs =
    choose (0, length xs - 1) >>= \i -> return $ xs !! i
```

## FsCheck

Similarly, FsCheck defines the type `gen` as a [computation expression](https://msdn.microsoft.com/en-us/library/dd233182.aspx).

The `choose` function is non-generic; instead it generates a [32-bit signed integer](https://msdn.microsoft.com/en-us/library/system.int32(v=vs.110).aspx?cs-save-lang=1&cs-lang=fsharp#code-snippet-1), with a uniform distribution in the closed interval *[l, h]*.

```fsharp
val choose : l:int * h:int -> Gen<int>
```

### Example: Take a random element from a list

```fsharp
open FsCheck
open FsCheck.Gen

let takeFromList xs =
    gen { let! i = choose (0, List.length xs - 1)
          return xs |> Seq.nth i }
```

Notice that `takeFromList` has equivalent signature with the one in Haskell:

``` haskell
val takeFromList :  xs:'a list -> Gen<'a> // F#
    takeFromList :: [a]        -> Gen a   // Haskell
```

### References

* QuickCheck
  * [QuickCheck: A Lightweight Tool for Random Testing of Haskell Programs](http://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf)
  * [QuickCheck v.0.2 manual](http://www.cse.chalmers.se/~rjmh/QuickCheck/manual.html)
  * [Test.QuickCheck](https://hackage.haskell.org/package/QuickCheck-2.7.6/docs/Test-QuickCheck.html)
  * [Random values of various types](https://hackage.haskell.org/package/random-1.0.1.1/docs/System-Random.html#t:Random)
* FsCheck
  * [Test data: generators, shrinkers and Arbitrary instances](https://fsharp.github.io/FsCheck/TestData.html)
