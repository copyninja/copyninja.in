Releasing debvulns: CLI for listing Debian vulnerabilities
##########################################################

:date: 2026-06-21 17:36 +5:30
:slug: debvulns-cli
:tags: generative-ai, debsecan, cli, debian
:author: copyninja
:summary: Announcing the release of debvulns CLI, a standalone utility built on the debsecan-mcp core logic.

Following up on my previous `post
<https://copyninja.in/blog/debsecan-mcp-pypi.html>`_, I have released the
`debvulns` `CLI <https://pypi.org/project/debsecan-mcp/>`_. This utility uses
the same parsing logic as the `debsecan-mcp` server but exposes the
functionality directly via the command line.

Why a new CLI?
==============

While Debian's native `debsecan` utility exists, it lacks modern output formats
like JSON and CSV, and fails to expose a significant amount of metadata
available in the Debian Security Team's daily snapshot.

Additionally, running a persistent Model Context Protocol (MCP) server
introduces context window overhead. The manifests and tool descriptions required
by the protocol consume tokens even when idle. For `debsecan-mcp`, the `MCP
Inspector utility
<https://modelcontextprotocol.io/docs/tools/inspector#pypi-package>`_ shows an
overhead of roughly 150 tokens.

By contrast, an LLM can parse a standard CLI help menu on-demand without
permanently draining the context window. Integrating the CLI into a persistent
agent workflow can be achieved via a skill file, allowing the LLM to leverage the
tool without repeated discovery overhead.

What else is NEW?
=================

During testing, I observed discrepancies between the output of `debsecan-mcp`/`debvulns`
and native `debsecan`. Debugging with an LLM revealed a bug in the version
`comparison logic
<https://github.com/copyninja/debsecan-mcp/commit/04e9990f2d7b2d85fe04f21c4f2e22fbc9aae365>`_
that caused `debvulns` to underreport vulnerabilities. This has been resolved.

The current interface supports structured formatting and customizable data backends:

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
        -sort-by {package,cve}
                              Sort vulnerabilities by 'package' or 'cve'
        --vuln-url VULN_URL   Custom URL or local path for Debian Security Tracker data
        --epss-url EPSS_URL   Custom URL or local path for EPSS scores data
        --suite SUITE         Debian suite name (e.g. bookworm, sid). Auto-detected by default.
        --cache-dir CACHE_DIR
                              Directory to cache fetched and parsed data (default: /var/cache/debvulns)
        --no-cache            Do not use cached data, force downloading and parsing
        -v, --verbose         Enable verbose debug logging (sent to stderr)

By allowing users to override data sources with local snapshots of the Debian
Security Tracker and EPSS feeds, `debvulns` can run natively in airgapped
environments.

What Next?
==========

The next step is building a Prometheus exporter for this vulnerability data to
streamline scanning and monitoring across data center infrastructure. Stay tuned.
