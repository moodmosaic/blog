---
layout: basic
title: Using class libraries from different platforms
---

<p>There are times where we need to use a library written in a different platform than the one we are currently on.&#0160;Most of the times a port already exists and we can choose to go with that.&#0160;Before using a port there are some good considerations:</p>
<ul>
<li>The size of the original codebase.</li>
<li>The kind of project (is it a logger, an&#0160;ORM, etc).</li>
<li>The current version of the original compared with the ported one. It depends on the activity of the original but if the ported is left 1+ year behind this is not a good sign.</li>
<li>The quality of the ported codebase,&#0160;framework usage, coding conventions. Does it use recommeded guidelines for the target framework?</li>
<li>The activity of the ported project. If it&#39;s low that&#39;s a sign that future versions might now show up. However, consider contributing.</li>
</ul>
<p>Besides ported code, there are cases where we can use a tool that allows us to use the original library from a different platform. For example, when working in .NET we can use Java class libraries through IKVM.NET which has a bytecode compiler called&#0160;<a href="http://www.ikvm.net/userguide/ikvmc.html" target="_blank" title="IKVM.NET Bytecode Compiler (ikvmc.exe)">IKVMC</a>.</p>

