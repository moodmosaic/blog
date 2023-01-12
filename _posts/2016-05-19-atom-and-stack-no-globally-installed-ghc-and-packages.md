---
layout: basic
title: Atom + Stack = no globally installed GHC/packages
---

When doing Haskell programming

* [Stack](http://www.haskellstack.org/) keeps my operating system clean of globally installed compilers and packages
* [atom-haskell](https://atom.io/users/atom-haskell) allows me to use [ghc-mod](https://hackage.haskell.org/package/ghc-mod), [hlint](https://hackage.haskell.org/package/hlint), and [stylish-haskell](https://hackage.haskell.org/package/stylish-haskell) without leaving the editor

## Stack

 * [Get Stack](http://docs.haskellstack.org/en/stable/README/#how-to-install)
 * Go to some folder (could be empty, or an existing project)
 * Create a [`Stack.yaml`](http://docs.haskellstack.org/en/stable/yaml_configuration/) file with just a line: `resolver: lts-5.17` or later, as shown in [https://www.stackage.org/lts/](https://www.stackage.org/lts/)
 * Execute `stack setup`

 The output should look like this:

```
$ stack setup
Downloaded lts-5.17 build plan.
Caching build plan
Fetching package index ...remote: Counting objects: 213332, done.
remote: Compressing objects: 100% (171913/171913), done.
Receiving objects: 100% (213332/213332), 49.84 MiB | 1.79 MiB/s, done.-reused 0

Resolving deltas: 100% (56842/56842), completed with 1 local objects.
From https://github.com/commercialhaskell/all-cabal-hashes
 * [new tag]         current-hackage -> current-hackage
Fetched package index.
Populated index cache.
Preparing to install GHC to an isolated location.
This will not interfere with any system-level installation.
ghc-7.10.3:   15.89 MiB / 128.40 MiB ( 12.37%) downloaded...

GHC installed to [..]\AppData\Local\Programs\stack\x86_64-windows\ghc-7.10.3\
stack will use a locally installed GHC
For more information on paths, see 'stack path' and 'stack exec env'
To use this GHC and packages outside of a project, consider using:
stack ghc, stack ghci, stack runghc, or stack exec
```
 * In the same directory as `Stack.yaml` execute `stack build ghc-mod hlint stylish-haskell`

## atom-haskell

 * Install [Atom](https://atom.io/)
 * Install [atom-haskell](https://atom.io/users/atom-haskell) via [apm](https://github.com/atom/apm): `apm install language-haskell ide-haskell ide-haskell-cabal haskell-ghc-mod autocomplete-haskell haskell-pointfree haskell-debug`
* Build `stylish-haskell`, `ghc-mod`, `pointfree` and `pointful`:
```
stack build stylish-haskell
stack build ghc-mod
stack build pointfree
stack build pointful
```
* Toggle `haskell-pointfree` via `ctrl+alt+shift+f`
* Toggle `haskell-debug` via `ctrl-shift-p`. To break on exceptions, launch the command palette, type in set break on exception, press enter and select the appropriate option.

That’s it - Happy Haskell programming!
