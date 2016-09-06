---
layout: post
title: All syntax extensions from witiko/markdown backported to jgm/lunamark and merged
tags:
  - programming
  - TeX
excerpt_separator: EOF
---

During the course of this year, I have been working on [witiko/markdown][] as a
part of a [project][] funded by the Masaryk University in Brno, Czech Republic.
The project is based on the [jgm/lunamark][] codebase and as of today, all the
bugfixes ([17][]) and syntax extensions ([19][], [20][]) introduced in
witiko/markdown were merged into jgm/lunamark. This effectively concludes the
project.

 [witiko/markdown]: https://github.com/Witiko/markdown "A package for converting and rendering Markdown documents inside TeX"
 [jgm/lunamark]: https://github.com/jgm/lunamark "Lua library for conversion between markup formats"
 [project]: http://www.fi.muni.cz/research/student_projects/projects.xhtml.cs#muni33_122015 "Schválené projekty k financování"
 [17]: https://github.com/jgm/lunamark/pull/17 "Non-found footnote ref no longer outputs unescaped text."
 [19]: https://github.com/jgm/lunamark/pull/19 "Added support for fenced code blocks."
 [20]: https://github.com/jgm/lunamark/pull/20 "Added support for Pandoc-style citations."
