debsecan-mcp v0.1.2 released to PyPI
####################################

:date: 2026-06-07 18:19 +5:30
:slug: debsecan-mcp-pypi
:tags: generative_ai, debsecan, mcp, debian
:author: copyninja
:summary: Brief about the release of debsecan-mcp server v0.1.2

I finally carved out some time today to prepare and release debsecan-mcp `v0.1.2
<https://pypi.org/project/debsecan-mcp/>`_ to PyPI. During this release, I
integrated PyPI's trusted publisher mechanism, which authenticates directly via
GitHub Actions and eliminates the need for manual uploads or static API tokens.

What is New?
============

There are no feature updates in this release; the changes are strictly focused 
on PyPI publishing requirements. This was handled entirely within the Antigravity 
IDE. 

The primary change replaces the `python-apt` dependency with `python-debian` for 
version comparison. PyPI rejects packages that reference external Git repositories, 
and `python-apt` lacks an official PyPI release. The original `python-apt` logic 
remains intact: if the system has `python-apt` installed, the server defaults to 
it. Otherwise, it falls back to the comparison logic implemented via the 
`python-debian` `NativeVersion` class.

What Next?
==========

The next release will introduce a standalone CLI utility called `debvulns`. It 
mirrors `debsecan` functionality but surfaces the cleaner, richer vulnerability 
data already implemented in `debsecan-mcp`. The code is written, and I will 
release it once testing is complete.

I also owe a post explaining my rationale for designing a CLI utility alongside 
the MCP server, and my broader thoughts on CLI vs. MCP workflows. I aim to publish 
that next week.
