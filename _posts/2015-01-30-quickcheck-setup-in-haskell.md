---
layout: basic
title: QuickCheck setup in Haskell
summary:
id-image: 8
tags:
    - Haskell
    - QuickCheck
---

Cabal is the build system for Haskell, it also doubles as a package manager. Create a *.cabal* file:

``` haskell
-- samples-quickcheck.cabal

Name:                   samples-quickcheck
Version:                0.0.0
Cabal-Version:          >= 1.10
Build-Type:             Simple

Library
  Default-Language:     Haskell2010
  HS-Source-Dirs:       tests
  GHC-Options:          -Wall
  Build-Depends:        base
                      , hspec
                      , QuickCheck

Test-Suite spec
  Type:                 exitcode-stdio-1.0
  Default-Language:     Haskell2010
  Hs-Source-Dirs:       tests
  Ghc-Options:          -Wall
  Main-Is:              Spec.hs
  Build-Depends:        base
                      , hspec
                      , QuickCheck
```

Enable automatic [Hspec](http://hspec.github.io/) discovery, so that each module
ending with *Spec* can be automatically considered as a test module:

``` haskell
-- tests/Spec.hs
{-# OPTIONS_GHC -F -pgmF hspec-discover #-}
```

Add a simple (icebreaker) test, because there's always a bit of work involved in
getting everything up and running:

``` haskell
-- tests/GettingStartedSpec.hs
module GettingStartedSpec (spec) where

import Test.Hspec
import Test.Hspec.QuickCheck

spec :: Spec
spec = do
    describe "icebreaker" $ do
        prop "icebreaker" $ \x y ->
            add x y == add y x

add :: Int -> Int -> Int
add x y = x + y
```

Create a Cabal sandbox, install dependencies, build the package, and enable the
tests, by running the following commands in a terminal:

``` bash
cabal update                      # Download the most recent list of packages
cabal install cabal-install       # Optional, can be prompted by cabal update

cabal sandbox init                # Initialise the sandbox
cabal install --only-dependencies # Install dependencies into the sandbox
cabal build                       # Build your package inside the sandbox

cabal configure --enable-tests    # Enable the test suites
```

Run the tests via Cabal's built-in test runner:

``` text
cabal test

Building samples-quickcheck-0.0.0...
Preprocessing library samples-quickcheck-0.0.0...
In-place registering samples-quickcheck-0.0.0...
Preprocessing test suite 'spec' for samples-quickcheck-0.0.0...
[1 of 2] Compiling GettingStartedSpec (tests\GettingStartedSpec.hs, dist\buil...
[2 of 2] Compiling Main (tests\Spec.hs, dist\build\spec\spec-tmp\Main.o)...
Linking dist\build\spec\spec.exe ...
Running 1 test suites...
Test suite spec: RUNNING...
Test suite spec: PASS
Test suite logged to: dist\test\samples-quickcheck-0.0.0-spec.log
1 of 1 test suites (1 of 1 test cases) passed.
```

See the test failing, as a form of [Double Entry Bookkeeping](http://c2.com/cgi/wiki?DoubleEntryBookkeeping):


``` haskell
-- Change:
add x y == add y x

-- To:
add x y == add 0 1
```

``` text
cabal test

Building samples-quickcheck-0.0.0...
Preprocessing library samples-quickcheck-0.0.0...
In-place registering samples-quickcheck-0.0.0...
Preprocessing test suite 'spec' for samples-quickcheck-0.0.0...
[1 of 2] Compiling GettingStartedSpec (tests\GettingStartedSpec.hs,...
[2 of 2] Compiling Main (tests\Spec.hs, dist\build\spec\spec-tmp\Main.o)...
Linking dist\build\spec\spec.exe ...
Running 1 test suites...
Test suite spec: RUNNING...

GettingStarted
  icebreaker
    icebreaker FAILED [1]

Failures:

  1) GettingStarted.icebreaker icebreaker
       Falsifiable (after 1 test and 1 shrink):
       0
       0

Randomized with seed 1531351472

Finished in 0.0000 seconds
1 example, 1 failure
Test suite spec: FAIL
Test suite logged to: dist\test\samples-quickcheck-0.0.0-spec.log
0 of 1 test suites (0 of 1 test cases) passed.
```

To use QuickCheck interactively, start a REPL by doing:

``` text
cabal repl spec
```

In the above command, `spec` is just the name of the test-suite, as declared in the .cabal file.


*For convenience, all the above are also [available on GitHub](https://github.com/moodmosaic/quickcheck-fscheck-samples/tree/master/samples-quickcheck).*

### References

* [The Haskell Cabal](https://www.haskell.org/cabal/)
* [What I Wish I Knew When Learning Haskell](http://www.stephendiehl.com/what/#cabal)
* [Automatic Hspec discovery](http://hspec.github.io/hspec-discover.html)
* [Icebreaker Test concept](http://blog.ploeh.dk/2015/01/10/diamond-kata-with-fscheck/)
* [Cabal User Guide](https://www.haskell.org/cabal/users-guide/)
 * [Developing with Sandboxes](https://www.haskell.org/cabal/users-guide/installing-packages.html#developing-with-sandboxes)
 * [Running test suites](https://www.haskell.org/cabal/users-guide/developing-packages.html#running-test-suites)
