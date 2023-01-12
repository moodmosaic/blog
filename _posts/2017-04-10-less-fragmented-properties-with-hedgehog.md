---
layout: basic
title: Less fragmented properties with Hedgehog
---

In this post, I'll be porting a property-based test from QuickCheck 2 and from FsCheck 2 to Hedgehog. See [this](/2017/04/09/property-based-testing-becomes-easier-with-hedgehog/) post for a brief introduction to Hedgehog.

### Setup phase (de)fragmentation

When writing a property, essentially we test a unit in isolation and so some of the [practices for traditional unit-tests](http://xunitpatterns.com/) can also apply to property-based tests. One common practice is to structure each test in three distinct parts executed in sequence

1. Setup phase
2. Exercise phase
3. Verification phase

In classic TDD these phases are called Arrange, Act, Assert, while in BDD they're called Given, When, Then.

### Examples in Haskell

Often the setup phase can become fragmented, as shown in the following example ([source](https://github.com/ploeh/dependency-rejection-samples/blob/9c214f403973dc6bd03083a23b934db2a9b13aaa/Haskell/MaitreDTests.hs#L36-L48)) usually because of the 'noise' that's generated because of the Arbitrary required by [`forAll`](https://hackage.haskell.org/package/QuickCheck-2.9.2/docs/src/Test-QuickCheck-Property.html#forAll):

![](http://nikosbaxevanis.com/images/articles/2017-04-10-less-fragmented-properties-with-hedgehog-1.png)

Notice how the setup phase becomes less fragmented with Hedgehog, as shown in the following example ([source](https://github.com/moodmosaic/dependency-rejection-samples/blob/80906b670f8a761586b3376cfe46dd232d59bd1f/Haskell/MaitreDTests.hs#L37-L49)):

![](http://nikosbaxevanis.com/images/articles/2017-04-10-less-fragmented-properties-with-hedgehog-2.png)

It can be worth pointing out also the fine-grained control we can have over the scope of generated values with the [Range combinators](https://hackage.haskell.org/package/hedgehog/docs/Hedgehog-Range.html)[^1].

### Examples in F#

FsCheck is similar to QuickCheck, so its `forAll` needs an Arbitrary as well, as shown below ([source](https://github.com/ploeh/dependency-rejection-samples/blob/9c214f403973dc6bd03083a23b934db2a9b13aaa/FSharp/BookingApi/Ma%C3%AEtreDTests.fs#L18-L31)):

![](http://nikosbaxevanis.com/images/articles/2017-04-10-less-fragmented-properties-with-hedgehog-3.png)

Here's the same example written with Hedgehog ([source](https://github.com/moodmosaic/fsharp-hedgehog-samples/blob/master/Hedgeploration.fsx#L38-L50)):

![](http://nikosbaxevanis.com/images/articles/2017-04-10-less-fragmented-properties-with-hedgehog-4.png)

Notice how the setup phase becomes less fragmented with Hedgehog, and how the `property` Computation Expression makes it easier to separate each distinct phase of the property-based test.

---

[^1]: The Range combinators aren't available yet in the F# version. They are tracked though by [this](https://github.com/hedgehogqa/fsharp-hedgehog/issues/85) GitHub item.
