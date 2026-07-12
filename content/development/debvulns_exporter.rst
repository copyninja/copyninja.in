Releasing debvulns-exporter:  Prometheus exporter for Debian System Vulnerabilities
###################################################################################

:date: 2026-06-21 16:00 +5:30
:slug: debvulns-exporter
:tags: debvulns, cli, prometheus, debian
:author: copyninja
:summary: Announcing release of debvulns-exporter, a standalone prometheus
          exporter for exporting vulnerabilities on Debian system


Following up on my previous `post
<https://copyninja.in/blog/debvulns-cli.html>`_ I'm now releasing
`debvulns-exporter`, a prometheus exporter for exporting Debian system
vulnerabilities. The vulnerability related logic remains same as earlier MCP
server and CLI utility.

Why Exporter?
=============

In my work I've been dealing with Debian and vulnerability management. Most
corporate environments use paid vulnerability management systems like
Tenable/Rapid5 etc. Of course these vulnerability management systems do have
additional capability but what I did not find is a OSS tool which can be used
for this purpose. `debsecan` was there but it was not having a presentable
format which can be dashboarded for consumption by management. The original
project started as MCP server for learning purpose but then I wondered why not
convert it to a prometheus exporter. Prometheus is defactor metrics platform
used across industry and hence this idea was born.

Design and Exported Metrics
===========================

The exporter designed as native prometheus exporter using `prometheus-client`
library. It has 2 threads one does fetching of vulnerability, epss data and
getting list of installed packages to find vulnerabilities on system and other
which exports these metrics. The full design and the exported metrics can be
found in this `design doc
<https://github.com/copyninja/debsecan-mcp/blob/main/docs/prometheus_exporter_design.md>`_.
The doc was created by brainstorming with *Claude Sonnet 4.6 Thinking* on
*Antrigravity* before getting into implementation part.

Testing and Dashboarding
========================

To test this I downloaded a old *Debian 11* and *Debian 12* cloud images from
`Debian Cloud team <https://cloud.debian.org/images/cloud/>`_. The older one was
picked make sure I get some set of vulnerabilities available on the system. The
setup looks like below image

.. image:: {static}/images/debvulns-exporter-setup.png

Since I'm not good at using Grafana I used the Antigravity with Claude Sonnet
4.6 to come up with dashboard and it did a decent job. This is a dashboard on
above local setup

.. image:: {static}/images/debvulns-exporter-dashboard.png

The exported dashboard which can be readily available is also part of `debvulns`
`source code
<https://github.com/copyninja/debsecan-mcp/blob/main/contrib/grafana/debvulns-dashboard.json>`_. 

Renaming the project
====================

To avoid conflict with debsecan which is already present on Debian I decided to
rename the project as *debvulns* with cli carrying the same name and exporter as
*debvulns-exporter* and mcp as *debvulns-mcp*. The release indicating the move
has been uploaded to `PyPi <https://pypi.org/project/debsecan-mcp/>`. The new
project now lives at `debvulns <https://pypi.org/project/debvulns/>`_.

Conclussion
===========

I don't know how useful it will be for others but this is just a idea which came
to me and which I felt be useful. I will consider packaging it to Debian both
CLI and exporter MCP will probably be shipped as artifact as I don't see much
use for it. Till then happy hacking.
