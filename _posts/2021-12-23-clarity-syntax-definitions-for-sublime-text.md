---
layout: basic
title: Clarity syntax definitions for Sublime Text
summary:
id-image: 3
tags:
    - Clarity
    - Stacks
---

Sublime Text currently ships without Clarity syntax definitions. Here's the process I followed to make my own [.sublime-syntax](https://www.sublimetext.com/docs/syntax.html) files.

## Prerequisites

- Git ^2.33.0
- Node.js ^16.13.1
- NPM ^8.1.2
- VS Code ^1.63.2
- Sublime Text ^4126

## Steps

The [syntax definitions](https://github.com/hirosystems/clarity-lsp/blob/a1e2808e13a8c4de4993389eb409d7c359248bcd/editors/code/syntaxes/clarity.yaml) for Clarity that I found online are maintained by Hiro, targeting VS Code and shipped as [TextMate grammars](https://code.visualstudio.com/api/language-extensions/syntax-highlight-guide).

<span>1. Clone the reference syntax definitions:</span>

```bash
git clone https://github.com/hirosystems/clarity-lsp
cd clarity-lsp
# Optionally, go to the exact same commit I used.
git checkout a1e2808e13a8c4de4993389eb409d7c359248bcd
cd editors/code/syntaxes
# We will be explicitly converting this afterwards.
rm clarity.tmLanguage.json
```

<span>2. Convert `clarity.yaml` to `clarity.json`:</span>

```bash
npm install js-yaml@4.1.0
# Convert:
#   yaml TextMate grammar ->
#   json TextMate grammar
npx js-yaml clarity.yaml > clarity.json
```

<span>3. Convert `clarity.json` to `clarity.tmLanguage`:</span>

```bash
# https://github.com/Togusa09/vscode-tmlanguage
code --install-extension Togusa09.tmlanguage
code clarity.json
# Then, use the 'Convert to tmLanguage File' command, save.
```

<span>4. Convert `clarity.tmLanguage` to `clarity.sublime-syntax`:</span>

```bash
subl clarity.tmLanguage
# Then, use the 'Convert Syntax to .syblime-syntax` command, save.
```

<span>5. Define the file extensions this syntax should be used for. Add the following to `clarity.sublime-syntax`:</span>

```
file_extensions:
  - clar
```

For convenience, here's the diff after adding it:

```diff
--- a/Data/Packages/User/clarity.sublime-syntax
+++ b/Data/Packages/User/clarity.sublime-syntax
@@ -2,6 +2,8 @@
 ---
 # http://www.sublimetext.com/docs/syntax.html
 name: clarity
+file_extensions:
+  - clar
 scope: source.clar
 contexts:
   main:
```

## Download

Get [clarity-sublime-syntax](https://github.com/moodmosaic/clarity-sublime-syntax) on github.
