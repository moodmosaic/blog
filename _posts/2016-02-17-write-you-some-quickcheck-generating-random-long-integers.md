---
layout: basic
title: Write you some QuickCheck - Generating random long integers
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

Two generators are required for this, which have been ported already on previous posts:

* a [generator for integers](/2016/02/13/write-you-some-quickcheck-generating-random-integers/) (with absolute value bounded by the generation size)
* a [generator that applies a function](/2016/02/10/write-you-some-quickcheck-generating-random-bytes/) (in this case [`Operators.int64<^T>`](https://msdn.microsoft.com/en-us/library/ee370563.aspx)) to an existing generator

The long integers are going to be generated by the integer returned by the first generator and then multiplied by a 16-bit[^2] integer's largest possible value.

Given the above, a long integer generator can be written as:

```fsharp
/// <summary>
/// Generates a 64-bit integer (with absolute value bounded by the generation
/// size multiplied by 16-bit integer's largest possible value).
/// </summary>
let int64 = Gen.int |> Gen.map (fun n -> Operators.int64 (n * 32767))

val int64 : Gen<int64>
```

Finally, here are some sample long integers:

```fsharp
> Gen.int64 |> Gen.generate;;
val it : int64 = -294903L

> Gen.int64 |> Gen.generate;;
val it : int64 = -32767L

> Gen.int64 |> Gen.generate;;
val it : int64 = 163835L

> Gen.int64 |> Gen.generate;;
val it : int64 = 131068L

> Gen.int64 |> Gen.generate;;
val it : int64 = -65534L

> Gen.int64 |> Gen.generate;;
val it : int64 = 229369L

> Gen.int64 |> Gen.generate;;
val it : int64 = 393204L
```

---

[^1]: From [Wikipedia](https://goo.gl/WRpzHU).
[^2]: To be more precise, I should use a 32-bit integer's largest possible value, but then the generated numbers become very big and less interesting.
