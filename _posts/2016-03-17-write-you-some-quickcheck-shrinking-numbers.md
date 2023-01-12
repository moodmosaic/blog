---
layout: basic
title: Write you some QuickCheck - Shrinking numberss
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In this post I'll be porting the function from QuickCheck that shrinks numbers.

## Prelude

When reporting back the actual input that caused a function to fail, QuickCheck first cleans the noise by reporting back the counterexample. — This is called *shrinking*. See how it works [here](http://www.dcc.fc.up.pt/~pbv/aulas/tapf/slides/quickcheck.html#shrinking).

All the shrinking functions have the same signature:

``` haskell
shrink :: a -> [a]
```

So in F# it can look like this:

```fsharp
'a -> 'a seq
```

## Shrinking numbers

QuickCheck shrinks towards smaller numeric values with the following generic function:

``` haskell
-- | Shrink an integral number.
shrinkIntegral :: Integral a => a -> [a]
shrinkIntegral x =
  nub
  $
  [ -x | x < 0, -x > x ]
  ++
  [ x' | x' <- takeWhile (<< x) (0:[ x - i | i <- tail (iterate (`quot` 2) x) ]) ]
 where
   -- a << b is "morally" abs a < abs b, but taking care of overflow.
   a << b = case (a >= 0, b >= 0) of
            (True , True ) -> a < b
            (False, False) -> a > b
            (True , False) -> a + b < 0
            (False, True ) -> a + b > 0
```
([View on Hackage](https://hackage.haskell.org/package/QuickCheck-2.8.2/docs/src/Test-QuickCheck-Arbitrary.html#shrinkIntegral))

Let's try this one out first:

``` text

λ> :t shrinkIntegral
shrinkIntegral :: Integral a => a -> [a]

λ> shrinkIntegral 1
[0]

λ> shrinkIntegral (-1)
[1,0]

λ> shrinkIntegral (-2)
[2,0,-1]

λ> shrinkIntegral (-3)
[3,0,-2]

λ> shrinkIntegral (-4)
[4,0,-2,-3]

λ> shrinkIntegral (-2)
[2,0,-1]

λ> shrinkIntegral 2
[0,1]

λ> shrinkIntegral 3
[0,2]

λ> shrinkIntegral 4
[0,2,3]

λ> shrinkIntegral (-700)
[700,0,-350,-525,-613,-657,-679,-690,-695,-698,-699]

λ> shrinkIntegral 700
[0,350,525,613,657,679,690,695,698,699]

λ> shrinkIntegral 7
[0,4,6]

λ> shrinkIntegral (-7)
[7,0,-4,-6]

```

Here's the equivalent one in F#:

```fsharp
(* This has the type n:int -> seq<int> but we'll
   come and fix this later on. *)

let shrinkNumber n =
    n
    |> Seq.unfold (fun s -> Some(n - s, s / 2))
    |> Seq.tail
    |> Seq.append [ 0 ]
    |> Seq.takeWhile (fun el -> abs n > abs el)
    |> Seq.append (if n < 0 then Seq.singleton -n
                   else Seq.empty)
    |> Seq.distinct
```

I'm going to try this one with the same values as the ones used in Haskell:

``` text

> val shrinkNumber : n:int -> seq<int>

> shrinkNumber 1;;
val it : seq<int> = seq [0]

> shrinkNumber (-1);;
val it : seq<int> = seq [1; 0]

> shrinkNumber (-2);;
val it : seq<int> = seq [2; 0; -1]

> shrinkNumber (-3);;
val it : seq<int> = seq [3; 0; -2]

> shrinkNumber (-4);;
val it : seq<int> = seq [4; 0; -2; -3]

> shrinkNumber (-2);;
val it : seq<int> = seq [2; 0; -1]

> shrinkNumber 2;;
val it : seq<int> = seq [0; 1]

> shrinkNumber 3;;
val it : seq<int> = seq [0; 2]

> shrinkNumber 4;;
val it : seq<int> = seq [0; 2; 3]

> shrinkNumber (-700);;
val it : seq<int> = seq [700; 0; -350; -525; ...]

> shrinkNumber 700;;
val it : seq<int> = seq [0; 350; 525; 613; ...]

> shrinkNumber 7;;
val it : seq<int> = seq [0; 4; 6]

> shrinkNumber (-7);;
val it : seq<int> = seq [7; 0; -4; -6]

```

## Shrinking not only integers, but floats, decimals, and so on

Here is again the `shrinkNumber` number function that was ported from Haskell:

```fsharp
(* This has the type n:int -> seq<int> but we'll
   come and fix this later on. *)

let shrinkNumber n =
    n
    |> Seq.unfold (fun s -> Some(n - s, s / 2))
    |> Seq.tail
    |> Seq.append [ 0 ]
    |> Seq.takeWhile (fun el -> abs n > abs el)
    |> Seq.append (if n < 0 then Seq.singleton -n
                   else Seq.empty)
    |> Seq.distinct
```

Because of `2`, and `0`, the F# compiler infers that this function works with integers.

Fortunately, there's a way to make this polymorphic with [compile-time generics](http://stackoverflow.com/a/4738404):

* Replace `0` with [GenericZero](https://msdn.microsoft.com/en-us/library/ee370581.aspx)
* Replace `2` with [GenericOne](https://msdn.microsoft.com/en-us/library/ee353503.aspx) + [GenericOne](https://msdn.microsoft.com/en-us/library/ee353503.aspx)
* Apply the [inline](https://msdn.microsoft.com/en-us/library/dd548047.aspx) modifier to the function

Here's the end result:

```fsharp
open FSharp.Core.LanguagePrimitives

let inline shrinkNumber n =
    let genericTwo = GenericOne + GenericOne
    n
    |> Seq.unfold (fun s -> Some(n - s, s / genericTwo))
    |> Seq.tail
    |> Seq.append [ GenericZero ]
    |> Seq.takeWhile (fun el -> abs n > abs el)
    |> Seq.append (if n < GenericZero then Seq.singleton -n
                   else Seq.empty)
    |> Seq.distinct
```

And now the type has become generic:

```fsharp
// Formatted, in order to be easier to read:
val inline shrinkNumber :
  n: ^a -> seq< ^a>
    when  ^a         : (static member get_Zero :           ->  ^a) and
          ^a         : (static member ( - )    :  ^a *  ^a ->  ^a) and
        ( ^a or  ^b) : (static member ( / )    :  ^a *  ^b ->  ^a) and
          ^a         : (static member Abs      :  ^a       ->  ^a) and
          ^a         : (static member ( ~- )   :  ^a       ->  ^a) and
          ^a         : comparison                                  and
          ^c         : (static member get_One  :           ->  ^c) and
        ( ^c or  ^d) : (static member ( + )    :  ^c *  ^d ->  ^b) and
          ^d         : (static member get_One  :           ->  ^d)
```

Let's try this one out:

``` text

> shrinkNumber 3.14;;
val it : seq<float> = seq [0.0; 1.57; 2.355; 2.7475; ...]

> shrinkNumber 4.0f;;
val it : seq<float32> = seq [0.0f; 2.0f; 3.0f; 3.5f; ...]

>shrinkNumber (-700.15M);;
val it : seq<decimal> = seq [700.15M; 0M; -350.075M; -525.1125M; ...]

>shrinkNumber 900s;;
val it : seq<int16> = seq [0s; 450s; 675s; 788s; ...]

```
