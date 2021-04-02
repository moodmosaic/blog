---
layout: basic
title: Write you some QuickCheck - Generating random integers
summary:
id-image: 7
tags:
    - QuickCheck
    - FSharp
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In the [previous](/2016/02/12/write-you-some-quickcheck-generating-random-booleans/) post I've generated random booleans. In this post, I'll be generating random 32-bit signed integers.

Recall that the `Gen<'a>` type is ported already in [this](/2016/02/09/write-you-some-quickcheck-prelude/) previous post as:

```fsharp
/// <summary>
/// A generator for values of type 'a.
/// </summary>
type Gen<'a> =
    private
    | Gen of (int -> StdGen -> 'a)
```

This means we can write functions that [handle state (—*monadster!*)](http://fsharpforfunandprofit.com/posts/monadster/) combining:

 * a **size** measure for data structures, i.e. list length, **numbers**, i.e. absolute bounds
 * a pseudo-random generation seed — more about that later on

So, to generate numbers, we need:

* a generator that picks numbers with absolute value bounded by the size
* a generator that gives us the current size at runtime

Porting Gen's `sized`

```fsharp
/// <summary>
/// Used to construct generators that depend on the size parameter.
/// </summary>
/// <param name="g">A generator for values of type 'a.</param>
let sized g =
    Gen(fun n r ->
        let (Gen m) = g n
        m n r)
```

Given the above, a 32-bit signed integer generator can be written as:

```fsharp
let int = Gen.sized (fun n -> Gen.choose (-n, n))

val int : Gen<int>
```

Here are some sample integers:

```fsharp
> Gen.int |> Gen.generate;;
val it : int = 0

> Gen.int |> Gen.generate;;
val it : int = -18

> Gen.int |> Gen.generate;;
val it : int = 0

> Gen.int |> Gen.generate;;
val it : int = 6

> Gen.int |> Gen.generate;;
val it : int = -6

> Gen.int |> Gen.generate;;
val it : int = 1

> Gen.int |> Gen.generate;;
val it : int = -2

> Gen.int |> Gen.generate;;
val it : int = 5

> Gen.int |> Gen.generate;;
val it : int = -4

> Gen.int |> Gen.generate;;
val it : int = -18

> Gen.int |> Gen.generate;;
val it : int = 5

> Gen.int |> Gen.generate;;
val it : int = -17

> Gen.int |> Gen.generate;;
val it : int = -7

> Gen.int |> Gen.generate;;
val it : int = -3

> Gen.int |> Gen.generate;;
val it : int = 1

> Gen.int |> Gen.generate;;
val it : int = 12

> Gen.int |> Gen.generate;;
val it : int = 5

> Gen.int |> Gen.generate;;
val it : int = 6

> Gen.int |> Gen.generate;;
val it : int = -8

> Gen.int |> Gen.generate;;
val it : int = -10

```

But there's a pattern here (can you tell?); all the generated integers are in the range [-30, 30] — that's because `generate` [sets the size for the generators to be up to 30](/2016/02/09/write-you-some-quickcheck-prelude/)[^1].

**Generating numbers in range [-999, 999]**


I can run the integer generator again, but this time I'll **override** the runtime **size**.

This prompts me to port Gen's `resize`

```fsharp
/// <summary>
/// Overrides the size parameter. Returns a generator which uses the given size
/// instead of the runtime-size parameter.
/// </summary>
/// <param name="n">The size that's going to override the runtime-size.</param>
let resize n (Gen m) = Gen(fun _ r -> m n r)
```

Finally, here are some sample integers in the range [-999, 999]:

```fsharp
> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = -808

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = 797

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = -676

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = 3

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = 304

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = -476

> Gen.int |> Gen.resize 999 |> Gen.generate;;
val it : int = 276

```

---

[^1]: That's how the generate function works in QuickCheck 2; It's slightly different (and easier to use) compared to QuickCheck 1.
