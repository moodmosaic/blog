---
layout: basic
title: Fantomas F# formatter in Visual Studio 2019
---

You can run Fantomas as an external tool from inside Visual Studio 2019 by using the Tools menu.

This technique should be compatible with

* Visual Studio 2015
* Visual Studio 2017
* Visual Studio 2019 (tested with 16.4.5)

### Add Fantomas to the Tools menu in Visual Studio 2019

1. Clone and build the source code on your local
```bash
git clone git@github.com:fsprojects/fantomas.git fantomas && cd $_
dotnet tool restore
dotnet paket restore
dotnet fake run build.fsx
```
2. Open the [external tools](https://docs.microsoft.com/en-us/visualstudio/ide/managing-external-tools?view=vs-2019) dialog box by choosing **Tools** > **External Tools**
3. Click **Add**, and then fill in the information<br>![](https://nikosbaxevanis.com/images/articles/2020-03-01-vs-external-command-fantomas-1.png)
4. You can also assign shortcut keys to it by choosing **Tools** > **Options**, expand **Environment**, and then choose **Keyboard** ![](https://nikosbaxevanis.com/images/articles/2020-03-01-vs-external-command-fantomas-2.png)

## Appendix

### JetBrains Rider

The [fsharp-support](https://github.com/JetBrains/fsharp-support) uses fantomas under the hood to format the source code. No need for any additional plugins.

### Visual Studio Code

The recommended way to use Fantomas is by using the [Ionide plugin](http://ionide.io/). Fantomas is integrated in [FSAutoComplete](https://github.com/fsharp/FsAutoComplete/) which is the language server used by Ionide.

Alternatively, you can install the [fantomas-fmt](https://marketplace.visualstudio.com/items?itemName=paolodellepiane.fantomas-fmt) extension.
