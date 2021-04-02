---
layout: basic
title: Short intro to QuickCheck
summary:
id-image: 1
tags:
    - QuickCheck
    - FSharp
---

**1. Write a function**

```fsharp
let ``the reverse of reverse of a list is the list itself`` =
    fun input -> input |> List.rev |> List.rev = input
```

Here's what that function does, in steps:

``` text
// Takes a list of integers as input
[1, 2, 3]

// Reverses the list
[3, 2, 1]

// Reverses the list again
[1, 2, 3]

// Returns true if the initial list equals the final one
[1, 2, 3] = [1, 2, 3]

True
```

**2. Generate random input**

Given a hypothetical `Gen` module, we can do something like this:

```fsharp
let generator = Gen.int |> Gen.list
// val generator : Gen<int list>
```

This is a generator for values of type `int list`[^1]. If we run it, it'll generate values like these:

``` text
[], [7], [34, -5, 3]
```

**3. Pass that random input to that function**


Now, we need a way to pass each one of these generated values to our function, and report back success or failure[^2]: essentially, that's what **QuickCheck** does.

See also: [Write you some QuickCheck](/2016/02/08/write-you-some-quickcheck/).

---

[^1]: QuickCheck contains a lot of basic generators, and some combinators for making custom generators.

[^2]: When reporting back the actual input that caused a function to fail, QuickCheck first cleans the noise — it tries to report back the *minimal* counter-example.
