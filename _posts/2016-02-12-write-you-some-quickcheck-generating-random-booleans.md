---
layout: basic
title: Write you some QuickCheck - Generating random booleans
summary:
id-image: 7
tags:
    - QuickCheck
    - FSharp
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In the [previous](/2016/02/11/write-you-some-quickcheck-generating-random-characters/) post I've generated random characters. In this post, I'll be doing the same for booleans.

Two generators are required for this, which have been ported already on previous posts:

* a generator that [lifts a value to the elevated world](http://fsharpforfunandprofit.com/posts/elevated-world/#return) of generators; that's the one ported as `Gen.init`
* a generator that randomly uses one of the above generators, that's `Gen.oneof`

Given the above, a boolean generator can be written as:

```fsharp
let bool =
    Gen.oneof [ Gen.init true
                Gen.init false ]

val bool : Gen<bool>
```

Finally, here are some sample booleans:

```fsharp
> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = true

> Gen.bool |> Gen.generate;;
val it : bool = false

> Gen.bool |> Gen.generate;;
val it : bool = false

```
