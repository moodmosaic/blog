---
layout: basic
title: Setting up Haskell on Windowss
---

Why do that and not use <a href="https://www.haskell.org/platform/">Haskell Platform</a> instead?

From what I have experienced so far, Haskell Platform falls behind the latest GHC release, and ships with Packages that <a href="http://www.reddit.com/r/haskell/comments/2al3vx/how_do_you_avoid_the_cabal_hell/">one might not need</a>.

Essentially there are two things needed:

* [GHC](https://www.haskell.org/ghc/) - Haskell's compiler and interactive environment.
* [Cabal](https://www.haskell.org/cabal/) - Haskell's build system, which also doubles as a package manager.

## GHC

* Download GHC for Windows from [here](https://downloads.haskell.org/~ghc/).
* Extract GHC contents to a folder and add to PATH:
  * \ghc-<em>x.y.z</em>\bin

## Cabal

* Find out [here](https://ghc.haskell.org/trac/ghc/wiki/Commentary/Libraries/VersionHistory) which version of Cabal matches the GHC version you downloaded
* Download Cabal for Windows from [here](https://www.haskell.org/cabal/release/)
  * You need to find a MinGW version, e.g. cabal-install-<em>x.y.z</em>-x86_64-unknown-mingw32
* Extract Cabal executable to a folder and add that folder to PATH
* Open a command prompt (cmd.exe) and execute:
  * cabal-<em>x.y.z</em>.exe `install`
  * cabal-<em>x.y.z</em>.exe `update`

And *that's it*!

--

## Did it work out?

Open a command prompt (cmd.exe) window or a Git Bash window and execute:

``` bash
$ which ghc
/c/ghc-x.y.z-x86_64-unknown-mingw32/ghc-x.y.z/bin/ghc

$ which cabal
/c/cabal-install-x.y.z-x86_64-unknown-mingw32/cabal
```

In the same command prompt executing `ghci` should launch GHCi (Haskell's interactive environment):

``` text
GHCi, version x.y.z: http://www.haskell.org/ghc/  :? for help
Loading package ghc-prim ... linking ... done.
Loading package integer-gmp ... linking ... done.
Loading package base ... linking ... done.
>
```

To get the `>` character show up in the prompt, add the following to `C:\Users\[YOUR USERNAME]\AppData\Roaming\ghc\ghci.conf`:

``` text
:set prompt "> "
```

That's it - Happy Haskell Programming (on Windows)!
