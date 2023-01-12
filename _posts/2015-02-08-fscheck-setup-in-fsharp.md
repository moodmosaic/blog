---
layout: basic
title: FsCheck setup in F#
---

In the previous [post](http://blog.nikosbaxevanis.com/2015/01/30/quickcheck-setup-in-haskell/) I used Cabal to bootstrap QuickCheck with Hspec. In Haskell, Cabal is a build system which also doubles as a package manager.

This post attempts to do the same in F#; bootstrap FsCheck with xUnit.net, but instead of MSBuild and XML, I'm going to use [Paket](http://fsprojects.github.io/Paket/index.html), [Fake](http://fsharp.github.io/FAKE/), and F#.


**Install Paket, if not already installed**:

```fsharp
// samples-fscheck.fsx

open System
open System.IO

Environment.CurrentDirectory <- __SOURCE_DIRECTORY__

if not (File.Exists "paket.exe") then
    let url = "https://github.com/fsprojects/Paket/releases/download/0.26.3/paket.exe"
    use wc = new Net.WebClient()
    let tmp = Path.GetTempFileName()
    wc.DownloadFile(url, tmp)
    File.Move(tmp, Path.GetFileName url)
```

**Reference Paket and specify the required packages**:

```fsharp
// samples-fscheck.fsx

#r "paket.exe"

Paket.Dependencies.Install """
    source https://nuget.org/api/v2
    nuget FsCheck.Xunit
    nuget FAKE
    nuget xunit.runners
""";;
```

**Reference Fake and specify the build parameters**:

```fsharp
// samples-fscheck.fsx

#r "packages/FAKE/tools/FakeLib.dll"

open Fake
open Fake.FscHelper

let outputPath = Path.Combine(Environment.CurrentDirectory, "bin")
let outputFile = @"bin\GettingStarted.dll";
let references =
    [ "packages/xunit/lib/net20/xunit.dll";
      "packages/FsCheck/lib/net45/FsCheck.dll";
      "packages/FsCheck.Xunit/lib/net45/FsCheck.Xunit.dll" ]
```

**Use Fake's DSL to define the build tasks**:

```fsharp
// samples-fscheck.fsx

Target "CreateOutputPath" (fun _ ->
    CreateDir outputPath)

Target "CleanOutputPath" (fun _ ->
    CleanDir outputPath)

Target "CompileFiles" (fun _ ->
    [ "tests/GettingStarted.fs" ]
    |> Fsc (fun options ->
        { options with
            Output = outputFile
            FscTarget = Library
            References = references }))

Target "CopyAssemblyReferences" (fun _ ->
    CopyFiles outputPath references)

Target "RunTests" (fun _ ->
    !! outputFile
    |> xUnit (fun options -> options))
```

**Compose the build tasks into a build pipeline**:

```fsharp
// samples-fscheck.fsx

"CreateOutputPath"
    ==> "CleanOutputPath"
    ==> "CompileFiles"
    ==> "CopyAssemblyReferences"
    ==> "RunTests"

RunTargetOrDefault "RunTests"
```

**Icebreaker**:

Add a simple (icebreaker) test, because there's always a bit of work involved in
getting everything up and running:

```fsharp
// tests/GettingStarted.fs

module GettingStarted

open System
open FsCheck.Xunit

let add x y = x + y

[<Property>]
let ``icebreaker`` x y =
    add x y = add y x
```

**Run a build**:

Run the following command in a terminal:

``` text
fsi samples-fscheck.fsx
```

See the tests running, and passing, via xUnit.net's built-in test runner:

``` text
fsi samples-fscheck

found: <Path>\samples-fscheck\paket.dependencies
Resolving packages:
    - exploring FAKE 3.14.9
    - exploring xunit 1.9.2
    - exploring xunit.runners 1.9.2
    - exploring FsCheck 1.0.4
    - exploring FsCheck.Xunit 1.0.4
Locked version resolutions written to <Path>\samples-fscheck
\paket.lock
Building project with version: LocalBuild
Shortened DependencyGraph for Target RunTests:
<== RunTests
<== CopyAssemblyReferences
<== CompileFiles
<== CleanOutputPath
<== CreateOutputPath

The resulting target order is:
 - CreateOutputPath
 - CleanOutputPath
 - CompileFiles
 - CopyAssemblyReferences
 - RunTests
Starting Target: CreateOutputPath
<Path>\samples-fscheck\bin already exists.
Finished Target: CreateOutputPath
Starting Target: CleanOutputPath (==> CreateOutputPath)
Deleting contents of <Path>\samples-fscheck\bin
Finished Target: CleanOutputPath
Starting Target: CompileFiles (==> CleanOutputPath)
FSC with args:[|"-o"; "bin\GettingStarted.dll"; "-a"; "--platform:anycpu";
  "--reference:packages/xunit/lib/net20/xunit.dll";
  "--reference:packages/FsCheck/lib/net45/FsCheck.dll";
  "--reference:packages/FsCheck.Xunit/lib/net45/FsCheck.Xunit.dll";
  "tests/GettingStarted.fs"|]
Finished Target: CompileFiles
Starting Target: CopyAssemblyReferences (==> CompileFiles)
Finished Target: CopyAssemblyReferences
Starting Target: RunTests (==> CopyAssemblyReferences)
<Path>\samples-fscheck\packages\xunit.runners\tools\xunit.co
nsole.clr4.exe "<Path>\samples-fscheck\bin\GettingStarted.dl
l"
xUnit.net console test runner (64-bit .NET 4.0.30319.34209)
Copyright (C) 2013 Outercurve Foundation.

xunit.dll:     Version 1.9.2.1705
Test assembly: <Path>\samples-fscheck\bin\GettingStarted.dll


1 total, 0 failed, 0 skipped, took 0.558 seconds
Finished Target: RunTests

---------------------------------------------------------------------
Build Time Report
---------------------------------------------------------------------
Target                   Duration
------                   --------
CreateOutputPath         00:00:00.0018707
CleanOutputPath          00:00:00.0046358
CompileFiles             00:00:03.4156059
CopyAssemblyReferences   00:00:00.0058707
RunTests                 00:00:01.9816487
Total:                   00:00:05.4737549
Status:                  Ok
---------------------------------------------------------------------
```

See the test failing, as a form of [Double Entry Bookkeeping](http://c2.com/cgi/wiki?DoubleEntryBookkeeping):


```fsharp
// Change:
add x y = add y x

// To:
add x y = add 0 1
```

Re-run the command in a terminal:

``` text
fsi samples-fscheck

xUnit.net console test runner (64-bit .NET 4.0.30319.34209)
Copyright (C) 2013 Outercurve Foundation.

xunit.dll:     Version 1.9.2.1705
Test assembly: <Path>\samples-fscheck\bin\GettingStarted.dll


GettingStarted.icebreaker [FAIL]

   Falsifiable, after 1 test (2 shrinks) (StdGen (1751373120,295969456)):
   (0, 0)
   Stack Trace:
   sorry no stacktrace

1 total, 1 failed, 0 skipped, took 0.679 seconds
Running build failed.
```

*For convenience, all the above are also [available on GitHub](https://github.com/moodmosaic/quickcheck-fscheck-samples/).*

### References

* [Don Syme's self-contained sample gist](https://gist.github.com/dsyme/9b18608b78dccf92ba33)
* [Fake](http://fsharp.github.io/FAKE/)
 * [Compiling F# Sources](http://fsharp.github.io/FAKE/fsc.html)
 * [FscHelper](http://fsharp.github.io/FAKE/apidocs/fake-fschelper.html)
 * [FileHelper](http://fsharp.github.io/FAKE/apidocs/fake-filehelper.html)
* [Paket](http://fsprojects.github.io/Paket/index.html)
* [Icebreaker Test concept](http://blog.ploeh.dk/2015/01/10/diamond-kata-with-fscheck/)
* [xUnit.net](https://github.com/xunit/xunit)
