---
layout: basic
title: Write you some QuickCheck - Prelude
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

I'll be basing this minimal F# version of QuickCheck on [QuickCheck 1.2.0.1](https://hackage.haskell.org/package/QuickCheck-1.2.0.1), the last version of QuickCheck depending exclusively on [base](https://hackage.haskell.org/package/base) and [Random 1.1](https://hackage.haskell.org/package/random-1.1)[^1].

Random is a basic random number generation library, including the ability to split random number generators. QuickCheck 1.2.0.1 uses

 * [StdGen](https://hackage.haskell.org/package/random-1.1/docs/src/System-Random.html#StdGen)
 * [newStdGen](https://hackage.haskell.org/package/random-1.1/docs/src/System-Random.html#newStdGen)
 * [randomR](https://hackage.haskell.org/package/random-1.1/docs/src/System-Random.html#randomIvalInteger)
 * [split](https://hackage.haskell.org/package/random-1.1/docs/src/System-Random.html#stdSplit)

from the Random package, and so it'll be easier to start by porting those in F#:

```fsharp
/// <summary>
/// This module deals with the common task of pseudo-random number generation.
/// It makes it possible to generate repeatable results, by starting with a
/// specified initial random number generator, or to get different results on
/// echch run by using the system-initialised generator or by supplying a seed
/// from some other source.
/// </summary>
/// <remarks>
/// This implementation uses the Portable Combined Generator of L'Ecuyer for
/// 32-bit computers, transliterated by Lennart Augustsson. It has a period of
/// roughly 2.30584e18.
/// </remarks>
[<AutoOpen>]
module internal LightCheck.Random

type StdGen =
    private
    | StdGen of int * int

/// <summary>
/// The next operation returns an Int that is uniformly distributed in the
/// rangge of at least 30 bits, and a new generator. The result of repeatedly
/// using next should be at least as statistically robust as the Minimal
/// Standard Random Number Generator. Until more is known about implementations
/// of split, all we require is that split deliver generators that are (a) not
/// identical and (b) independently robust in the sense just given.
/// </summary>
let private next (StdGen (s1, s2)) =
    let k    = s1 / 53668
    let k'   = s2 / 52774
    let s1'  = 40014 * (s1 - k * 53668)  - k * 12211
    let s2'  = 40692 * (s2 - k' * 52774) - k' * 3791
    let s1'' = if s1' < 0 then s1' + 2147483563 else s1'
    let s2'' = if s2' < 0 then s2' + 2147483399 else s2'
    let z    = s1'' - s2''
    let z'   = if z < 1 then z + 2147483562 else z

    (z', StdGen (s1'', s2''))

/// <summary>
/// The split operation allows one to obtain two distinct random number
/// generators. This is very useful in functional programs (for example, when
/// passing a random number generator down to recursive calls), but very little
/// work has been done on statistically robust implementations of split.
/// </summary>
let split (StdGen (s1, s2) as std) =
    let s1' = if s1 = 2147483562 then 1 else s1 + 1
    let s2' = if s2 = 1 then 2147483398 else s2 - 1
    let (StdGen (t1, t2)) = next std |> snd

    (StdGen (s1', t2), StdGen (t1, s2'))

/// <summary>
/// The range operation takes a range (lo,hi) and a random number generator g,
/// and returns a random value, uniformly distributed, in the closed interval
/// [lo,hi], together with a new generator.
/// </summary>
/// <remarks>
/// It is unspecified what happens if lo > hi. For continuous types there is no
/// requirement that the values lo and hi are ever produced, although they very
/// well may be, depending on the implementation and the interval.
/// </remarks>
let rec range (l, h) rng =
    if l > h then range (h, l) rng
    else
        let (l', h')    = (32767, 2147483647)
        let b           = h' - l' + 1
        let q           = 1000
        let k           = h - l + 1
        let magnitude   = k * q
        let rec f c v g =
            if c >= magnitude then (v, g)
            else
                let (x, g') = next g
                let v'      = (v * b + (x - l'))
                f (c * b) v' g'
        let (v, rng') = f 1 0 rng

        (l + v % k), rng'

let private r = int System.DateTime.UtcNow.Ticks |> System.Random

/// <summary>
/// Provides a way of producing an initial generator using a random seed.
/// </summary>
let createNew() =
    let s       = r.Next() &&& 2147483647
    let (q, s1) = (s / 2147483562, s % 2147483562)
    let s2      = q % 2147483398

    StdGen (s1 + 1, s2 + 1)
```

To port QuickCheck's basic generators, and combinators for making custom ones, we need a type of `Gen<'a>`:

```fsharp
/// <summary>
/// LightCheck exports some basic generators, and some combinators for making
/// new ones. Gen of 'a is the type for generators of 'a's and essentially is
/// a State Monad combining a pseudo-random generation seed, and a size value
/// for data structures (i.e. list length).
/// Using the type Gen of 'a, we can specify at the same time a set of values
/// that can be generated and a probability distribution on that set.
///
/// Read more about how it works here:
/// http://www.dcc.fc.up.pt/~pbv/aulas/tapf/slides/quickcheck.html#the-gen-monad
/// http://quviq.com/documentation/eqc/index.html
/// </summary>
module LightCheck.Gen

/// <summary>
/// A generator for values of type 'a.
/// </summary>
type Gen<'a> =
    private
    | Gen of (int -> StdGen -> 'a)
```

Finally, we also need a function[^2] that *runs* a generator; taking a `Gen<'a>` and returning random test data of type `'a`:

```fsharp
/// <summary>
/// Runs a generator. The size passed to the generator is up to 30; if you want
/// another size then you should explicitly use 'resize'.
/// </summary>
let generate (Gen m) =
    let (size, rand) = Random.createNew() |> Random.range (0, 30)
    m size rand

val generate : Gen<'a> -> 'a
```

We're all set! Let's generate some random test data in the [next post(s)](/2016/02/08/write-you-some-quickcheck/).

---

[^1]: The other thing to notice about QuickCheck 1.2.0.1, is that it's the last version before version 2 came out. In version 2 there's added support for [shrinking](http://www.dcc.fc.up.pt/~pbv/aulas/tapf/slides/quickcheck.html#shrinking), which is a major advancement. The shrinking functions are going to be ported from [QuickCheck 2.8.2](https://hackage.haskell.org/package/QuickCheck-2.8.2).

[^2]: The implementation of the `generate` function is based on both QuickCheck 1.2.0.1 and QuickCheck 2.8.2.
