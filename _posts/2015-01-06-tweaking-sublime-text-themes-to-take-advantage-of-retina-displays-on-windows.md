---
layout: basic
title: Tweaking Sublime Text themes to take advantage of Retina displays
summary:
id-image: 9
tags:
    - SublimeText
---

Sublime Text already supports Retina displays on OS X, however it seems that the story is currently a bit different on Windows.

**Update - Monday, November 14, 2016**: To make this a whole lot easier, I've forked [buymeasoda/soda-theme](https://github.com/buymeasoda/soda-theme) into [moodmosaic/soda-theme](https://github.com/moodmosaic/soda-theme). If you clone moodmosaic/soda-theme, it will show nicely on Retina displays on Windows.

## Problem

You experience fuzzy fonts when using Sublime Text with Retina displays on Windows, [even after updating to a newer version of Sublime Text](http://sublimetext.userecho.com/topic/106898-text-is-fuzzy-on-a-high-dpi-display-windows/).

## Manual solution

* Navigate to the theme's root folder
* Search for images matching the pattern `*@2x.*`
* Copy and paste each image, in-place, by removing the *@2x* part from the filename, thus replacing any existing one.

<div class="message"><p>The above technique has been <b>tested with</b> Sublime Text's <b>Default Theme</b>, and <b>Soda Theme</b> (both dark and light variants).</p><p>Soda Theme requires the following images to be excluded from the above process: tab-active, tab-hover, tab-inactive, tabset-background.</p></div>

## Automated solution

If done manually, this can take some time. - Here is a way to **automate this with F#**:

<div class="message"><p>If you don't have the F# compiler and runtime, install it from <a href="http://www.microsoft.com/en-us/download/details.aspx?id=41654" target="_">here</a>.</p><p>On a clean system without any edition of Visual Studio, the compiler, runtime, and F# Interactive will be deployed without IDE tooling.</p></div>

Save the following F# program to a file:

```fsharp
module Sublime.Text.Windows

open System
open System.IO

type Theme = { Path : string; DpiAgnostics : string list }

let patch theme =
    Directory.GetFiles(theme.Path, "*@2x.*", SearchOption.AllDirectories)
    |> Seq.filter
        (fun candidate ->
            theme.DpiAgnostics
            |> Seq.exists
                (fun agnostic ->
                    Path.GetFileNameWithoutExtension(candidate) = agnostic)
            |> not)
    |> Seq.iter
        (fun hiDpi ->
            let lowDpi = hiDpi.Replace("@2x", String.Empty)
            File.Delete lowDpi
            File.Copy(hiDpi, lowDpi))
    ()

```

<p class="message">Assuming the above program is saved as <code>Program.fs</code> in <code>X:\Tools</code>.</p>

* Before opening Sublime Text, add the compiler and F# Interactive executables' folder to PATH. - On my machine they are located in C:\Program Files (x86)\Microsoft SDKs\F#\x.x\Framework\vx.x.
* Open Sublime Text
* Install [SublimeREPL](https://sublimerepl.readthedocs.org/en/latest/).
* Go to Tools > SublimeREPl > F#
* In the F# Interactive prompt type, `#I @"X:\Tools"`, followed by `#load "Program.fs"`.

You should see something like:

```fsharp
namespace FSI_0002.Sublime.Text.Windows
  type Theme =
    {Path: string;
     DpiAgnostics: string list;}
  val patch : theme:Theme -> unit
```

Once you `open Sublime.Text.Windows`, execute the `patch` function by supplying a value for `theme`.

### Soda Theme

```fsharp
let sodaTheme = {
    Path = @"replace_this_with_the_soda_theme_path";
    DpiAgnostics = [
            "tab-active@2x"
          ; "tab-hover@2x"
          ; "tab-inactive@2x"
          ; "tabset-background@2x" ]
} // In one line for SublimeREPL.
```

```fsharp
patch sodaTheme
```

<h3>Default Theme</h3>

Default Theme is .zipped in `\Packages\Theme - Default.sublime-package` and has to be unzipped first, patched, and then zipped back again.

```fsharp
let defaultTheme = {
    Path = @"replace_this_with_the_unzipped_theme_path";
    DpiAgnostics = []
} // In one line for SublimeREPL.
```

```fsharp
patch defaultTheme
```

Enjoy Sublime Text with Retina displays on Windows!
