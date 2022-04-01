---
layout: basic
title: F# on Emacs with Spacemacs
summary:
id-image: 3
tags:
    - FSharp
---

While F# is first-class citizen in Visual Studio, there's also support for text editors:

 * via [fsharp/fsharpbinding](https://github.com/fsharp/fsharpbinding) for Emacs, Vim, and Sublime Text
 * via [Krzysztof-Cieslak/FSharp.Atom](https://github.com/Krzysztof-Cieslak/FSharp.Atom) for Atom

<div class="message"><p>There's also support for MonoDevelop and Xamarin Studio, which are IDEs. See the <a href="http://fsharp.org/guides/mac-linux-cross-platform/index.html#editing">Cross-Platform Development with F#</a> for more details.</p></div>

## Beyond Visual Studio

In my experience so far, getting F# to work outside Visual Studio [requires](https://github.com/fsharp/fsharpbinding/blob/master/emacs/README.md#installation) [quite](https://github.com/fsharp/fsharpbinding/blob/master/vim/README.mkd#installing) an [effort](https://github.com/fsharp/fsharpbinding/blob/master/sublimetext/README.md#general-steps).

There are always a couple of steps to be done, and basic knowledge of the target text editor is also assumed.

## A flexible, easy way, with Spacemacs

I learned Spacemacs in a [tweet by Chris Bowdon](https://twitter.com/nikosbaxevanis/status/588088768869072897). Spacemacs is essentially [an Emacs configuration](https://github.com/syl20bnr/spacemacs#batteries-included).

![Image](http://nikosbaxevanis.com/images/articles/2015-04-25-fsharp-on-emacs-with-spacemacs-05.png)

<div class="message">
  <p><strong>Emacs for Windows</strong>, is <a href="ftp://ftp.gnu.org/gnu/emacs/windows/">available</a>. Unzip to a folder and run. - An <em>.emacs.d</em> folder is automatically created in ~/AppData/Roaming/, storing all Emacs settings.</p>
</div>

### Install Spacemacs

Open a Git Bash command prompt window:


``` bash
# Backup any existing Emacs configuration.
mv ~/AppData/Roaming/.emacs.d ~/AppData/Roaming/.emacs.bak

# Clone Spacemacs in place of the old Emacs configuration.
git clone --recursive http://github.com/syl20bnr/spacemacs ~/AppData/Roaming/.emacs.d
```

Then, launch Emacs and Spacemacs will automatically load, installing all required packages. After that, Emacs must be restarted.

### Enable F#

After opening Emacs, it should show `~/.spacemacs` under Recent files. - Click on that file to open it.

Add F# as shown in the following diff-output:

``` diff
diff --git a/.spacemacs b/.spacemacs
index d6cedae..a47936c 100644
--- a/.spacemacs
+++ b/.spacemacs
@@ -23,6 +23,7 @@
      ;; markdown
      ;; org
      ;; syntax-checking
+     fsharp
      )
    ;; A list of packages and/or extensions that will not be install and loaded.
    dotspacemacs-excluded-packages '()
```

Then hit CTRL+C & CTRL+C, and the Spacemacs layer for F# will install itself.

### Screenshots

![Image](http://nikosbaxevanis.com/images/articles/2015-04-25-fsharp-on-emacs-with-spacemacs-04.png)

![Image](http://nikosbaxevanis.com/images/articles/2015-04-25-fsharp-on-emacs-with-spacemacs-01.png)

![Image](http://nikosbaxevanis.com/images/articles/2015-04-25-fsharp-on-emacs-with-spacemacs-02.png)

![Image](http://nikosbaxevanis.com/images/articles/2015-04-25-fsharp-on-emacs-with-spacemacs-03.png)

The F# projects shown in the above screenshots are [Qaiain](https://github.com/GreanTech/Qaiain) and [DiamondFsCheck](https://github.com/ploeh/DiamondFsCheck).
