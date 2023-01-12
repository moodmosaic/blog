---
layout: basic
title: Write you some QuickCheck - Generating random listss
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In this post I'll be generating lists of random length.

The maximum length of the list will depend on the runtime size parameter. See [this](/2016/02/13/write-you-some-quickcheck-generating-random-integers) previous post that explains how size works.

Two generators are required for this:

* a [generator that picks numbers](/2016/02/10/write-you-some-quickcheck-generating-random-bytes) with absolute value bounded by the size (that's Gen's `choose`)
* a generator that creates lists of a *given* length (that's Gen's `vector`). The length will be the number obtained by the above generator

Gen's `choose` is already available. Porting Gen's `vector`, which essentially [means](https://hackage.haskell.org/package/QuickCheck-2.8.2/docs/src/Test-QuickCheck-Gen.html#vectorOf) porting [replicateM](https://hackage.haskell.org/package/base-4.8.2.0/docs/Control-Monad.html#v:replicateM)

```fsharp
/// <summary>
/// Takes a list of generators of type 'a, evaluates each one of them, and
/// collect the result, into a new generator of type 'a list.
/// </summary>
/// <param name="l">The list of generators of type 'a.</param>
/// <remarks>
/// This is written so that the F# compiler will use a tail call, as shown in
/// the resulting excerpt of generated IL:
///   IL_0000: nop
///   IL_0001: call class [FSharp.Core]Microsoft.FSharp.Core.FSharpFunc`2<cl...
///   IL_0006: ldarg.0
///   IL_0007: call class [FSharp.Core]Microsoft.FSharp.Collections.FSharpLi...
///   IL_000c: call class LightCheck.Gen/Gen`1<!!0> LightCheck.Gen::'init'<c...
///   IL_0011: tail.
///   IL_0013: call !!1 [FSharp.Core]Microsoft.FSharp.Collections.ListModule...
///   IL_0018: ret
/// See also:
///   http://stackoverflow.com/a/6615060/467754,
///   http://stackoverflow.com/a/35132220/467754
/// </remarks>
let sequence l =
    let k m m' =
        Gen.bind m (fun x ->
            Gen.bind m' (fun xs ->
                Gen.init (x :: xs)))
    Gen.init [] |> List.foldBack k l

/// <summary>
/// Generates a list of the given length.
/// </summary>
/// <param name="n">The number of elements to replicate.</param>
/// <param name="g">The generator to replicate.</param>
let vector n g =
    Gen.sequence [ for _ in [ 1..n ] -> g ]
```

**Creating a custom `gen` computation expression**

Most, if not all, monadic computations on the `Gen<'a>` type can be written in terms of `Gen.bind` and `Gen.init` (return).

A computation expression just makes this a whole lot easier:

```fsharp
[<AutoOpen>]
module Builder =
    type GenBuilder() =
        member this.Bind       (g1, g2) = Gen.bind g1 g2
        member this.ReturnFrom (f)      = f

    let gen = GenBuilder()
```

Given the above, a generator for lists of random length can be written as:

```fsharp
/// <summary>
/// Generates a list of random length. The maximum length of the list depends
/// on the size parameter.
/// </summary>
/// <param name="g">The generator from which to create a list from.</param>
let list g = Gen.sized (fun s -> gen { let! n = Gen.choose (0, s)
                                       return! Gen.vector n g })
```

Finally, here are some sample lists of type `int` and `char`, but it can be *any* generator of type `'a` really

```fsharp
> Gen.int |> Gen.list |> Gen.generate;;
val it : int list = [-10; 1; 0; -14; 11; -2; -7]

> Gen.int |> Gen.resize 1337 |> Gen.list |> Gen.generate;;
val it : int list =
  [-1027; 739; -571; 475; 979; 144; -1239; -1282; -1012; -932; -952]

> Gen.char |> Gen.list |> Gen.generate
val it : char list = [':'; 'e'; '«']

> Gen.char |> Gen.list |> Gen.generate
val it : char list =
  ['H'; 'µ'; 'Ø'; '°'; 'ò'; 'k'; '<'; 't'; 't'; 'È'; '\\'; '\151'; 'Æ'; '¥';
   '/'; '\137'; 'é'; '\145'; '/'; 'Ì'; 'ï'; '\153'; '·'; '7']

```
