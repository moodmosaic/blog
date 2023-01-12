---
layout: basic
title: Why QuickCheck can be useful
---

One of the many challenges when doing Test-Driven Development is to come up with [good enough](http://blog.ploeh.dk/2009/03/05/ConstrainedNon-Determinism/) random input values, in such a way that the [system under test](http://xunitpatterns.com/SUT.html) (SUT) can be exercised every time though the same code path.

These random input values, that exercise the SUT every time through the same code path, belong into a category, or set, *or class*, named [Equivalence Class](http://xunitpatterns.com/equivalence%20class.html), after the [xUnit Test Patterns](http://xunitpatterns.com/) book terminology.

QuickCheck provides a DSL, through its various combinators, that allows you to create and consume such Equivalence Classes.

<p class="message">A practical use of Equivalence Classes can be found at the <a href="http://www.ndcvideos.com/#/app/video/2291">Equivalence Classes, xUnit.net, FsCheck, Property-Based Testing</a> NDC talk by <a href="http://blog.ploeh.dk/">Mark Seemann</a>.</p>

QuickCheck also provides a sophisticated mechanism for executing test cases multiple times, and doing analytics through the concepts of *classifying* and *shrinking*, when reporting back the test results.

<p class="message">In-depth introduction for QuickCheck can be found in the <a href="http://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf">original paper</a>, and for FsCheck can be found in <a href="http://fsharpforfunandprofit.com/posts/property-based-testing/">this</a> article by <a href="http://fsharpforfunandprofit.com/">Scott Wlaschin</a>.</p>

QuickCheck is created by [Koen Claessen](http://www.cse.chalmers.se/~koen/) and [John Hughes](http://en.wikipedia.org/wiki/John_Hughes_(computer_scientist)). FsCheck is created by [Kurt Schelfthout](http://fortysix-and-two.blogspot.com/) as a port of QuickCheck.
