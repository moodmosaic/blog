---
layout: basic
title: Creating F# lists with AutoFixtures
---

## Problem

An exception is thrown when requesting F# lists with AutoFixture:

```fsharp
Fixture().Create<int list>()

(* ObjectCreationException: AutoFixture was unable to create an instance
 * of type [..] because the traversed object graph contains a circular
 * reference.
 *     Path:
 *       Microsoft.FSharp.Collections.FSharpList`1[System.Int32] tail -->
 *       Microsoft.FSharp.Collections.FSharpList`1[System.Int32] -->
 *       Microsoft.FSharp.Collections.FSharpList`1[System.Int32]
 *)
```

## Diagnosis

Lists, and other collection types in the [System.Collections.Generic](https://msdn.microsoft.com/en-us/library/system.collections.generic.aspx) namespace, can be *filled* via the [constructor that takes an `IEnumerable<T>` as a parameter](https://msdn.microsoft.com/en-us/library/fkbw11z0.aspx).

AutoFixture creates such instances [via the constructor with the less number of parameters](http://blog.ploeh.dk/2009/03/24/HowAutoFixtureCreatesObjects.aspx)[^1].

[F# lists](https://msdn.microsoft.com/en-us/library/dd233224.aspx) and the collection types in the System.Collections.Generic namespace are different. One of the differences is the way they're created:

* F# lists can be built using the functions in the [List module](https://msdn.microsoft.com/en-us/library/ee353738.aspx).
* Collection types in the System.Collections.Generic namespace can be built via the aforementioned constructor(s).

## Customize AutoFixture to work with F# lists

The following customization can be applied to a `Fixture` instance, in order to support the creation of F# lists:

```fsharp
module TestConventions

open System
open Ploeh.AutoFixture
open Ploeh.AutoFixture.Kernel

let (|List|_|) candidate =
    match candidate |> box with
    | :? Type as t when t.IsGenericType &&
                        t.GetGenericTypeDefinition() = typedefof<list<_>>
        -> Some <| List(t.GetGenericArguments().[0])
    | _ -> None

type ReflectiveList =
    static member Build<'a>(args : obj list) =
        [ for a in args do
                yield a :?> 'a ]
    static member BuildTyped t args =
        typeof<ReflectiveList>
            .GetMethod("Build")
            .MakeGenericMethod([| t |])
            .Invoke(null, [| args |])

let listBuilder (f : IFixture) =
    { new ISpecimenBuilder with
            member this.Create(request, ctx) =
                match request, ctx with
                | List t, _ ->
                    [ for i in 1..f.RepeatCount -> ctx.Resolve t ]
                    |> ReflectiveList.BuildTyped t
                | _ -> box <| NoSpecimen() }

[<Sealed>]
type ListCustomization() =
    interface ICustomization with
        member this.Customize fixture =
            fixture.Customizations.Add <| listBuilder fixture
```

## Typical usage

```fsharp
let fixture = Fixture().Customize(ListCustomization())

let listOfInts = fixture.Create<int list>()
(* Length = 3
 *     [0]: 221.0
 *     [1]: 207.0
 *     [2]: 129.0
 *)

let listOfStrings = fixture.Create<string list>()
(* Length = 3
 *     [0]: "224f149d-502d-4387-9203-b5e8a7b4e208"
 *     [1]: "4f2503f4-9bbe-4dd5-ad69-f0e482a41b45"
 *     [2]: "d973a186-ed30-4c0b-abfa-addc1ac7cc46"
 *)
```

## Declarative usage

The above can also be written declaratively using [AutoData](http://blog.ploeh.dk/2010/10/08/AutoDataTheoriesWithAutoFixture.aspx) theories. Here's an example for xUnit.net:

```fsharp
open Xunit.Extensions
open Ploeh.AutoFixture.Xunit

[<Sealed>]
type TestConventionsAttribute() =
    inherit AutoDataAttribute(
        Fixture().Customize(ListCustomization()))

[<Theory; TestConventions>]
let ``Request a list of ints`` (xs : int list) =
    (* Length = 3
     *     [0]: 234
     *     [1]: 245
     *     [2]: 3
     *)
    ()

[<Theory; TestConventions>]
let ``Request a list of strings`` (xs : string list) =
    (* Length = 3
     *     [0]: "ea2a2310-84dc-46e1-aa58-850c57175765"
     *     [1]: "69d4222b-c272-48e9-871c-a4b2715e1945"
     *     [2]: "58ba2b15-b33f-46b8-af91-a50f915c3581"
     *)
    ()
```

## Credits

* The fact that AutoFixture can't currently create F# lists out of the box was put to my attention by [Rune Ibsen](https://twitter.com/runeibsen).
* The customization in this post is based on a similar one that [Rune Ibsen](https://twitter.com/runeibsen) showed me.
* The `ReflectiveList` code is adapted from [this](http://www.fssnip.net/1L) snippet by [Rick Minerich](https://twitter.com/rickasaurus).

---

[^1]: This strategy can be [overridden](http://blog.ploeh.dk/2011/04/19/ConstructorstrategiesforAutoFixture/).
