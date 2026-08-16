Releasing debvulns-exporter and debvulns CLI 0.2.2
##################################################

:date: 2026-08-16 17:00 +5:30
:slug: debvulns-exporter-newversion
:tags: debvulns, cli, prometheus, debian
:author: copyninja
:summary: Announcing release debvulns 0.2.2 with enhancements and an updated
          dashboard


I made another minor release with several enhancements: handling non-Debian
origin vulnerabilities, improving data caching, and sharing the cache
between the ``debvulns`` CLI and the exporter. Additionally, there are a few
improvements on the dashboard front. Here is a breakdown of what changed.

Handling Vulnerabilities in Non-Debian Origin Packages
======================================================

During the previous release, I noticed that the ``grafana`` package—which is
not in Debian and was installed via an upstream repository—was reported as
vulnerable with multiple issues. Looking into why this happened, I found that
all the CVEs reported in the dashboard were indeed listed on
``security-tracker.debian.org``, but without a fixed version or status
description. The logic assumed no fix was available and marked the package
as vulnerable on the dashboard.

How Did I Solve This?
---------------------

Google maintains a distributed vulnerability database for open-source
projects called `osv.dev <https://osv.dev>`_. I checked the generic vulnerability
data for those CVEs on OSV (unbound to any specific distribution) and found that
the issues were already fixed in the upstream version I was running. What I
needed was a way to differentiate native Debian packages from non-Debian
packages, which corresponds to the ``Origin`` field in APT metadata.

Pitfall
-------

The AI-generated code initially attempted to differentiate package origin using
``apt_pkg.PackageRecords`` and its ``origin`` field. However, many native Debian
packages were incorrectly flagged as non-Debian. On closer inspection, when an
upgrade is available for a package, the installed version's origin field can be
unset. I had to resolve this by detecting available upgrades and inspecting the
candidate version's origin instead, which was implemented in `this patch
<https://github.com/copyninja/debsecan-mcp/commit/cc72d477d0d0a6ff2ba5d2b70007bfc13fd50172>`_.
This solution was proudly crafted by me ;-) (partly because I ran out of API
limits and had to wait 6 hours for the next reset).

Caching OSV Data
----------------

Initially, the AI implemented the exporter to re-download the entire OSV dataset
on every run, which was unnecessary. Since vulnerability data does not change
rapidly once published, caching it on disk for longer than the standard 24-hour
Debian/EPSS cache makes sense. OSV vulnerability data is now cached for 7 days
before a refresh is triggered.

All cache expiration thresholds remain configurable via CLI flags.

Catch
-----

One caveat with this approach: I have not yet verified whether every upstream CVE
is tracked on ``security-tracker.debian.org``. In the case of ``grafana``, the
entries existed. This feature operates on the assumption that
``security-tracker.debian.org`` indexes CVE metadata regardless of whether the
package is native to Debian. I plan to re-evaluate this and add fallback handling
if that assumption fails.


Unified Cache Directory for CLI and Exporter
============================================

Another issue was cache segregation: the ``debvulns`` CLI utility defaulted to
``/var/cache/debvulns``, while the Prometheus exporter used
``/var/cache/debvulns-exporter``. While harmless when running only one tool,
installing both led to duplicated cache storage and redundant network requests.
Since the core evaluation logic is identical across both tools, they now share a
unified cache directory to eliminate duplicate downloads.

Dashboard Changes
=================

During the initial dashboard rollout, my test environment (my laptop alongside
Debian 11 and Debian 12 VMs) reported a high aggregated vulnerability count.
It was not immediately obvious whether these were distinct vulnerabilities or the
same CVEs replicated across all three machines. This mirrors common questions
raised during vulnerability reviews:

- How many unique vulnerabilities are present across the fleet?
- Which unique packages are affected?

The dashboard has been redesigned to surface unique vulnerability counts
alongside affected package lists. The updated dashboard is shown below:

.. image:: {static}/images/new_debvulns_dashboard.png


What's Next?
============

A few planned items remain to make ``debvulns`` a comprehensive vulnerability
reporting toolkit for Debian systems:

1. **Kernel Vulnerability Handling:** Currently, installing a patched kernel
   marks the vulnerability as resolved, even if the system has not rebooted into
   it. The system remains exposed while the vulnerable kernel is executing in
   memory. Factoring in running kernel versions is crucial.
2. **Reboot and Service Restart Tracking:** Similar to kernel upgrades requiring
   a reboot, userland library and binary fixes require running services to be
   restarted. This is typically detected via ``needrestart``. Integrating this
   behavior directly into ``debvulns`` will provide complete visibility in a
   single dashboard metric.
3. **Debian Packaging:** Once the above features are stable, the final step is
   packaging ``debvulns`` for Debian so it can be installed directly from the
   archive.

Until then, happy hacking.
