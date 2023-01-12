---
layout: basic
title: Property-based testing becomes easier with Hedgehog
---

[Hedgehog](https://github.com/hedgehogqa) is a brand new property-based testing system.

In mainstream scenarios it

* [makes properties less fragmented](/2017/04/10/less-fragmented-properties-with-hedgehog/)
* doesn't require to write shrinkers; it does it for free
* provides fine-grained control over the scope and shrinking of generated values

It is available for

* [Haskell](http://hackage.haskell.org/package/hedgehog)
* [F#](https://www.nuget.org/packages/Hedgehog/)
* [R](https://cran.r-project.org/web/packages/hedgehog/index.html)
* [Scala](https://bintray.com/hedgehogqa/scala-hedgehog)

[Jacob Stanley](https://twitter.com/jacobstanley), the creator of Hedgehog, made yesterday an [announcement on Haskell Reddit](https://www.reddit.com/r/haskell/comments/646k3d/) and overall received interesting feedback, including [one from Koen Claessen](https://www.reddit.com/r/haskell/comments/646k3d/ann_hedgehog_property_testing/dg1485c/?st=j1bul7ft&sh=63ac4ad6)[^1].

---

[^1]: Koen Claessen created QuickCheck together with John Hughes.
