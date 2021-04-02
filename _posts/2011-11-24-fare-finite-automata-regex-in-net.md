---
layout : basic
title  : Fare - Finite Automata/Regex in .NET
summary: Generate random text matching a given regex
id-image: 1
tags:
    - AutoFixture
    - Fare
---

[Fare](https://github.com/moodmosaic/Fare "Fare - [F]inite [A]utomata and [R]egular [E]xpressions")
is an effort to bring a
[DFA](http://en.wikipedia.org/wiki/Deterministic_finite-state_machine "Deterministic finite-state machine")/[NFA](http://en.wikipedia.org/wiki/Nondeterministic_finite-state_machine "Nondeterministic finite-state machine")
(finite-state automata) implementation from Java to .NET. It was primarily created as an internal library in [AutoFixture](https://github.com/AutoFixture/AutoFixture) for [supporting the RegularExpressionAttribute](http://nikosbaxevanis.com/blog/2011/12/11/regularexpressionattribute-support-in-autofixture/) class.

Fare is a .NET port of the well established Java library
[dk.brics.automaton](http://www.brics.dk/automaton/ "dk.brics.automaton")
with API as close as possible to the corresponding dk.brics.automaton
classes.

Fare also includes a port of
[Xeger](http://code.google.com/p/xeger/ "A Java library for generating random text from regular expressions.") which
is a Java library for generating random text from regular expressions.
The latter is possible in .NET using the
[Rex](http://research.microsoft.com/en-us/projects/rex/ "Rex is a tool that explores .NET regexes and generates members efficiently.")
tool.

Source code on [GitHub](https://github.com/moodmosaic/Fare), binaries on [NuGet](https://www.nuget.org/packages/Fare/).
