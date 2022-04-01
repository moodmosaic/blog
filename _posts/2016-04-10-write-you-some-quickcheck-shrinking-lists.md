---
layout: basic
title: Write you some QuickCheck - Shrinking lists
summary:
id-image: 7
tags:
    - QuickCheck
    - FSharp
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In the [previous](/2016/03/17/write-you-some-quickcheck-shrinking-numbers) post I've ported the function from QuickCheck that shrinks numbers. In this post I'll be doing the same for lists.

## Prelude

QuickCheck shrinks lists by dropping elements from them, given a shrinking function for individual elements.

``` haskell

shrinkList :: (a -> [a]) -> [a] -> [[a]]

-- (a -> [a]) the shrinker of type `a`.
--       [a]  the input to be shrinked.
--      [[a]] the output shrinked list.

```

## Shrinking a list of integers

To shrink a list of integers, say `[1,2,3]` we need

 * a function that shrinks integers (*shrinker*)
 * a function that shrinks lists, applying for each element the shrinker

In Haskell QuickCheck 2.8.2, it looks like this:

``` haskell
-- A function that shrinks integers (the shrinker)
λ> :t shrinkIntegral
shrinkIntegral :: Integral a => a -> [a]

-- Examples
λ> shrinkIntegral 1
[0]

λ> shrinkIntegral 2
[0,1]

λ> shrinkIntegral 3
[0,2]

-- A function that shrinks lists (taking a shrinker and a list)
λ> :t shrinkList
shrinkList :: (a -> [a]) -> [a] -> [[a]]

-- Examples
λ> shrinkList shrinkIntegral [1]
[[],[0]]

λ> shrinkList shrinkIntegral [2]
[[],[0],[1]]

λ> shrinkList shrinkIntegral [3]
[[],[0],[2]]

λ> shrinkList shrinkIntegral [1,2]
[[],[2],[1],[0,2],[1,0],[1,1]]

λ> shrinkList shrinkIntegral [1,2,3]
[[],[2,3],[1,3],[1,2],[0,2,3],[1,0,3],[1,1,3],[1,2,0],[1,2,2]]
```

## Doing similar in F#

A possible shrinking function for lists in F# can be based on the following Haskell code snippet, taken from QuickCheck 2.8.2:

``` haskell

shrinkOne []       = []
shrinkOne (x : xs) = [ x' : xs  |  x' <-       shr  x ]
                  ++ [ x  : xs' | xs' <- shrinkOne xs ]
```

```fsharp
module Shrink

(* A shrinker for values of type 'a. *)
type Shrink<'a> =
    | Shrink of ('a -> 'a seq)

(* Shrinks a sequence of elements of type 'a. First it yields an empty
   sequence, and then it iterates the input sequence, and shrinks each
   one of the items given the shrinker which is passed as a parameter. *)
let shrinkList xs (Shrink shr) =
    let rec shrinkImp xs =
        match xs with
        | []       -> Seq.empty
        | (h :: t) ->
            seq {
                yield []
                for h' in        shr h  -> h' :: t
                for t' in (shrinkImp t) -> h  :: t'
            }
    shrinkImp xs
```

I'm going to try this one with the same values as the ones used in Haskell:

```fsharp

(* A function that shrinks integers (the shrinker) *)
> let shrinker = Shrink shrinkNumber;;
val shrinker : Shrink<int> = Shrink <fun:shrinker@3>

(* A function that shrinks lists (taking a list and a shrinker) *)
val shrinkList : xs:'a list -> Shrink<'a> -> seq<'a list>

(* Examples *)
> shrinker |> shrinkList [1];;
val it : seq<int list> = seq [[]; [0]]

> shrinker |> shrinkList [2];;
val it : seq<int list> = seq [[]; [0]; [1]]

> shrinker |> shrinkList [3];;
val it : seq<int list> = seq [[]; [0]; [2]]

> shrinker |> shrinkList [1; 2];;
val it : seq<int list> = seq [[]; [0; 2]; [1]; [1; 0]; ...]

> shrinker |> shrinkList [1; 2; 3];;
val it : seq<int list> = seq [[]; [0; 2; 3]; [1]; [1; 0; 3]; ...]

```

Not exactly the same results as in Haskell, but similar. That's because the `shrinkList` function in F# is based on a small part of the `shrinkList` function in Haskell.

For the record, here's the complete [`shrinkList` from QuickCheck 2.8.2](https://hackage.haskell.org/package/QuickCheck-2.8.2/docs/src/Test-QuickCheck-Arbitrary.html#shrinkList):

``` haskell
-- Shrinks a list of values given a shrinking function for individual values.
shrinkList :: (a -> [a]) -> [a] -> [[a]]
shrinkList shr xs =
  concat [ removes k n xs | k <- takeWhile (>0) (iterate (`div`2) n) ] ++ shrinkOne xs

 where
  n = length xs

  -- The F# version is based only on this function below:
  shrinkOne []       = []
  shrinkOne (x : xs) = [ x' : xs  | x'  <-       shr  x ]
                    ++ [ x  : xs' | xs' <- shrinkOne xs ]

  removes k n xs
    | k > n     = []
    | null xs2  = [[]]
    | otherwise = xs2 : map (xs1 ++) (removes k (n - k) xs2)
   where
    xs1 = take k xs
    xs2 = drop k xs
```

Hopefully, the [next post(s)](/2016/02/08/write-you-some-quickcheck/) will become more interesting where I'll be gluing together some generators, shrinkers, and properties.
