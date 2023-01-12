---
layout: basic
title: Write you some QuickCheck
---

In the next couple of posts, I'm going to implement a basic version of QuickCheck from scratch. Follow along and you'll:

 * learn how [QuickCheck](https://hackage.haskell.org/package/QuickCheck) works
 * become better at using it[^1]
 * see a bit of [Quviq's QuickCheck in Erlang](http://quviq.com/documentation/eqc/index.html) as well

I'll be porting tons of Haskell code in F#[^2]. — Disclaimer: I've [done](http://blog.nikosbaxevanis.com/2011/11/19/notes-from-porting-java-code-to-net/) [similar](http://blog.nikosbaxevanis.com/2011/11/24/fare-finite-automata-regex-in-net/) [things](http://blog.nikosbaxevanis.com/2015/09/25/regex-constrained-strings-with-fscheck/) in the past. It's fun!

Here's a list of posts to follow:

 * [Write you some QuickCheck: Prelude](/2016/02/09/write-you-some-quickcheck-prelude/)
 * [Write you some QuickCheck: Generating random bytes](/2016/02/10/write-you-some-quickcheck-generating-random-bytes/)
 * [Write you some QuickCheck: Generating random characters](/2016/02/11/write-you-some-quickcheck-generating-random-characters/)
 * [Write you some QuickCheck: Generating random booleans](/2016/02/12/write-you-some-quickcheck-generating-random-booleans/)
 * [Write you some QuickCheck: Generating random 32-bit signed integers](/2016/02/13/write-you-some-quickcheck-generating-random-integers/)
 * [Write you some QuickCheck: Generating random 64-bit signed integers](/2016/02/17/write-you-some-quickcheck-generating-random-long-integers/)
 * [Write you some QuickCheck: Generating random 32-bit floating-points](/2016/02/25/write-you-some-quickcheck-generating-random-floats/)
 * [Write you some QuickCheck: Generating random lists](/2016/02/19/write-you-some-quickcheck-generating-random-lists/)
 * [Write you some QuickCheck: Generating random strings](/2016/02/22/write-you-some-quickcheck-generating-random-strings/)
 * [Write you some QuickCheck: Shrinking numbers](/2016/03/17/write-you-some-quickcheck-shrinking-numbers/)
 * Write you some QuickCheck: Shrinking bytes, characters, and booleans
 * [Write you some QuickCheck: Shrinking lists](/2016/04/10/write-you-some-quickcheck-shrinking-lists/)
 * Write you some QuickCheck: Shrinking strings
 * [Write you some QuickCheck: Properties](/2016/05/27/write-you-some-quickcheck-properties/)
 * [Write you some QuickCheck: Properties that hold under certain conditions](/2016/06/13/write-you-some-quickcheck-properties-that-hold-under-certain-conditions/)
 * Write you some QuickCheck: Properties that are labeled
 * Write you some QuickCheck: Properties that are labeled conditionally
 * Write you some QuickCheck: Putting everything together
 * Write you some QuickCheck: Where's the Arbitrary class?

Source code on GitHub: [https://gist.github.com/moodmosaic/65c576732722b3b7a200](https://gist.github.com/moodmosaic/65c576732722b3b7a200)

---

[^1]: Basic knowledge of QuickCheck and property-based testing is assumed. Read this [post](http://fsharpforfunandprofit.com/posts/property-based-testing/) first by [Scott Wlaschin](http://fsharpforfunandprofit.com/), followed by this [Pluralsight course](https://www.pluralsight.com/courses/fsharp-property-based-testing-introduction) by [Mark Seemann](http://blog.ploeh.dk/).

[^2]: If you're just looking for a production-ready QuickCheck based tool in F#, use [FsCheck](https://github.com/fscheck/FsCheck).
