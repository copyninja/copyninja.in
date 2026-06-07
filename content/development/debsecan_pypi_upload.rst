debsecan-mcp v0.1.2 released to pypi
####################################

:date: 2026-06-07 18:19 +5:30
:slug: debsecan-mcp-pypi
:tags: generative_ai, debsecan, mcp, debian
:author: copyninja
:summary: Brief about release of debsecan-mcp server v0.1.2

Its been a while and I did not get time to work on debsecan-mcp and today took
some time to prepare and release debsecan-mcp version to PyPI. Learnt about
using PyPI trusted publisher mechanism which works completely based on Github
Actions and requires no manual upload or token during this release.

What is New?
============

There is no feature related changes in debsecan-mcp 0.1.2 its purely changes
done to upload to PyPI. Again completely done using new Antigravity IDE. It
added support to use `python-debian` in version comparison and replaced
`python-apt` from dependencies. This is done mainly because `python-apt` has no
PyPI release and we were referencing it from Git repository which PyPI rejects
during publishing. The code still has `python-apt` logic and if it detects
`python-apt` on system it continues to use that over the comparison logic
implemented using `python-debian` `NativeVersion` class.


What Next?
==========

I've been having some ideas to improve this utility and next release is going to
contain a CLI utility called `debvulns`. This is similar to `debsecan` but will
have much cleaner and richer information which we already introduced in
`debsecan-mcp`. Code is already created once I've tested it enough and having
guarantee that its working as expected I will release it for general use.

There is also a post pending on why CLI over MCP as per my own understanding and
also reason for me to design the CLI utility for debsecan-mcp. Hopefully I will
try to complete it by next week.
