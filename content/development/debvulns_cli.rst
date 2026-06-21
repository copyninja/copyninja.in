Releasing debvulns: CLI for listing Debian vulnerabilities
##########################################################

:date: 2026-06-21 17:36 +5:30
:slug: debvulns-cli
:tags: generative-ai, debsecan, cli, debian
:author: copyninja
:summary: Brief about debvulns cli based on debsecan-mcp

As mentioned in my previous `post
<https://copyninja.in/blog/debsecan-mcp-pypi.html>`_ I've today released the
debvulns CLI. A CLI utility using same logic as debsecan-mcp server but
providing data via command line.

Why new CLI?
============

Debian already has `debsecan` utility so why a new utility? Reason being
debsecan does not expose a lot of data which is available to it via Debain
Security Team's daily snapshot. It also does not provide newer format exports
like json and csv.

Secondly MCP has an overhead it eats up into your context window even when you
are not using it actively. This is the manifests and tool description which is
provided by the MCP. For `debsecan-mcp` this is not very huge when I last
checked it with `MCP Inspector utility
<https://modelcontextprotocol.io/docs/tools/inspector#pypi-package>`_ this was
around 150 tokens. When you provide the CLI LLM is smart enough to parse the
help of command to understand how to use it and note that it does not directly
eat into your context Window and only when you need to use tool it uses and you
can all the time create a skill file so that LLM knows how to do things without
needing to refiguring out everything again.

What else is NEW?
=================

I noticed that debsecan-mcp/debvulns output was not matching what debsecan was
producing. After letting LLM figure it out, turns out issue was in version
`comparison logic
<https://github.com/copyninja/debsecan-mcp/commit/04e9990f2d7b2d85fe04f21c4f2e22fbc9aae365>`_
wich made debvulns produce lower number of vulnerabilities than what debsecan
produced.

debvulns has following command line interface and can produce output in json as
well as CSV format.

.. code-block:: sh

    usage: debvulns [-h] [-s {critical,high,medium,low,negligible}] [-f {json,csv}] [--sort-by {package,cve}] [--vuln-url VULN_URL] [--epss-url EPSS_URL] [--suite SUITE]
                [--cache-dir CACHE_DIR] [--no-cache] [-v]

    debvulns - CLI Debian Vulnerabilities Tracker

    options:
        -h, --help            show this help message and exit
        -s, --severity {critical,high,medium,low,negligible}
                        Filter vulnerabilities by severity
        -f, --format {json,csv}
                        Output format (default: json)
        --sort-by {package,cve}
                        Sort vulnerabilities by 'package' or 'cve'
        --vuln-url VULN_URL   Custom URL or local path for Debian Security Tracker data
        --epss-url EPSS_URL   Custom URL or local path for EPSS scores data
        --suite SUITE         Debian suite name (e.g. bookworm, sid). Auto-detected by default.
        --cache-dir CACHE_DIR
                        Directory to cache fetched and parsed data (default: /var/cache/debvulns)
        --no-cache            Do not use cached data, force downloading and parsing
        -v, --verbose         Enable verbose debug logging (sent to stderr)

It is provided with ability to point to local snapshot of Security Tracker data
as well as EPSS vulnerability data. This will be useful to host this on
airgapped systems.

What Next?
==========

Next thing I'm already working on is a prometheus exporter for vulnerability
data. This will allow easier exposing of vulnerability data especially in
datacenter like environment. Cya till then.
