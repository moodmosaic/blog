---
layout: basic
title: Write you some QuickCheck - Properties that hold under certain conditions
---

This post is part of a [series of posts on implementing a minimal version of QuickCheck](/2016/02/08/write-you-some-quickcheck/) from scratch. The source code is [available on GitHub](https://gist.github.com/moodmosaic/65c576732722b3b7a200).

In the [previous](/2016/05/27/write-you-some-quickcheck-properties/) post, we've seen how an assertion can be turned into a property and sampled, using only the `Gen` and `Property` modules:

```fsharp
fun x -> if x > 3 then true else false
|> Property.forAll g
|> Property.evaluate
|> Gen.sample;;

val it : Result list =
  [ { Status = Some false
      Args   = ["0"] }

    { Status = Some false
      Args   = ["-1"] }

    { Status = Some false
      Args   = ["-1"] }

    { Status = Some false
      Args   = ["3"] }

    { Status = Some false
      Args   = ["1"] }

    { Status = Some false
      Args   = ["-5"] }

    { Status = Some true
      Args   = ["9"] }

    { Status = Some true
      Args   = ["12"] }

    { Status = Some true
      Args   = ["5"] }

    { Status = Some false
      Args   = ["-2"] }

    { Status = Some true
      Args   = ["6"] }
  ]
>
```

Notice that in this particular case, *some* tests passed:

```fsharp
val it : Result list =
  [ ...

    { Status = Some true
      Args   = ["9"] }

    { Status = Some true
      Args   = ["12"] }

    { Status = Some true
      Args   = ["5"] }

    ...

    { Status = Some true
      Args   = ["6"] }
  ]
>
```

while the rest of them failed:

```fsharp
val it : Result list =
  [ { Status = Some false
      Args   = ["0"] }

    { Status = Some false
      Args   = ["-1"] }

    { Status = Some false
      Args   = ["-1"] }

    { Status = Some false
      Args   = ["3"] }

    { Status = Some false
      Args   = ["1"] }

    { Status = Some false
      Args   = ["-5"] }

    ...

    { Status = Some false
      Args   = ["-2"] }

    ...
  ]
>
```

In the assertion:

```fsharp
fun x -> if x > 3 then true else false
```

when we feed `x` with an integer that is lower then `3`, the test fails.

In order to *discard* test cases where `x` is lower then `3`, we use the `==>` operator:

```fsharp
fun x -> x > 3 ==> if x > 3 then true else false
```

The `==>` operator takes a predicate and a function, and returns a property that holds when the predicate returns true:

```fsharp
(* Returns a property that holds under certain conditions. Laws which are
   simple equations are conveniently represented by boolean functions but
   sometimes laws hold only under certain conditions. So this implication
   combinator represents such conditional laws. *)
let implies b x =
    if b then x |> convert
    else     () |> convert

let (==>) b x = implies b x

let private convert candidate =
    match box candidate with
    | :? Property   as p -> p
    | :? bool       as b -> boolProperty b
    | :? Lazy<bool> as b -> boolProperty b.Value
(* ... *)
    | _                  -> unitProperty
```

## Sample output

```fsharp
fun x -> x > 3 ==> if x > 3 then true else false
|> Property.forAll g
|> Property.evaluate
|> Gen.sample;;

val it : Property.Result list =
  [ { Status = null (* Discarded *)
      Args   = ["0"] }

    { Status = null (* Discarded *)
      Args   = ["2"] }

    { Status = null (* Discarded *)
      Args   = ["-3"] }

    { Status = null (* Discarded *)
      Args   = ["-4"] }

   { Status = null (* Discarded *)
     Args   = ["0"] }

   { Status = Some true
     Args   = ["5"] }

   { Status = Some true
     Args   = ["4"] }

   { Status = null (* Discarded *)
     Args   = ["1"] }

   { Status = Some true
     Args   = ["6"] }

   { Status = Some true
     Args   = ["16"] }

   { Status = null (* Discarded *)
     Args   = ["-2"] }
  ]
>
```

## Tip: Using Lazy Computations

By default, F# expressions are evaluated *exactly once* when defined, however sometimes *at most once* semantics are needed, as shown below:

```fsharp
fun x -> x = x <> 0 ==> (1/x = 1/x)
|> Property.forAll g
|> Property.evaluate
|> Gen.sample

(* System.DivideByZeroException : Attempted to divide by zero. *)
```

In the above example, `x` can get the value `0`, and although the test case *is* discared, the right part of the condition is eagerly evaluated. — In such cases, a [lazy computation](https://msdn.microsoft.com/en-us/visualfsharpdocs/conceptual/lazy-computations-%5Bfsharp%5D) is required:

```fsharp
fun x -> x = x <> 0 ==> lazy (1/x = 1/x)
|> Property.forAll g
|> Property.evaluate
|> Gen.sample

(* Does not throw an exception. *)

val it : Property.Result list =
  [ { Status = null (* Discarded *)
      Args   = ["0"] }

   { Status = Some true
     Args   = ["1"] }

    { Status = Some true
      Args   = ["-3"] }

    { Status = Some true
      Args   = ["-7"] }

   { Status = Some true
     Args   = ["4"] }

   { Status = Some true
     Args   = ["9"] }

   { Status = Some true
     Args   = ["13"] }

   { Status = Some true
     Args   = ["-22"] }

   { Status = Some true
     Args   = ["16"] }

   { Status = Some true
     Args   = ["-37"] }
  ]
>
```
