---
layout: post
title: "Demystifying .NET Terminology"
date: 2024-06-26
description: |
  How does .NET Framework differ from .NET Core?  What is Entity Framework Core, and how is it related?
tags:
- .NET
- ASP.NET
---

So much modern technology is built with Microsoft's .NET, it seems inevitable that I would have to learn about it eventually.  When I did, I found so much of the terminology confusing; things had overlapping names and nothing seemed consistent.  This table hopefully helps you (and me!) keep it all straight.

<table>
<thead>
<th>Name</th>
<th>Description</th>
</thead>
<tbody>
<tr>
<td>.NET</td>
<td>
The "application platform" for all other technologies listed here.  It consists of:
<ul>
<li>The Common Language Runtime (or CLR)</li>
<li>Libraries (standard lib type stuff; i/o, dates and times, etc)</li>
<li>Compiler</li>
<li>SDK and tools</li>
<li>"App stacks" such as ASP.NET Core and Windows Forms</li>
</ul>
</td>
</tr>
<tr>
<td>.NET Framework</td>
<td>A closed-source, Windows-only version of .NET that is deprecated.  It's last version is .NET Framework 4.8.1, released in 2022.</td>
</tr>
<tr>
<td>.NET Core</td>
<td>A newer implementation of .NET that is <a href="https://github.com/dotnet">open source</a> and cross-platform (Windows, Linux, and macOS).  Since .NET Framework is deprecated, it is commonly referred to as just ".NET".<br><br>The latest version as of writing is .NET Core 8.0.</td>
</tr>
<tr>
<td>ASP.NET</td>
<td>ASP.NET is a web framework extending .NET for making web services.  The term can refer to the original ASP.NET, the closed-source, .NET Framework version which, naturally, is also deprecated, or ASP.NET Core, the newer, open-source version targeting .NET Core.</td>
</tr>
<tr>
<td>ASP.NET Core</td>
<td>The newer, <a href="https://github.com/dotnet/aspnetcore">open-source</a>, cross-platform web framework.</td>
</tr>
<tr>
<td>Entity Framework</td>
<td>An Object-Relational Mapping (ORM) library for .NET.  The story with Entity Framework is very similar to the story with ASP.NET; it was originally shipped as part of .NET Framework, but has been split off and has been superseded by Entity Framework Core.
</td>
</tr>
<tr>
<td>Entity Framework Core</td>
<td>The newer, <a href="https://github.com/dotnet/efcore">open-source</a>, cross-platform ORM.</td>
</tr>
</tbody>
</table>

I'll add more terms as I encounter them!
