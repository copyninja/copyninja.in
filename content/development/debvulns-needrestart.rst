Releasing debvulns with needrestart and kernel vulnerability handling
#####################################################################

:date: 2026-11-01 18:03 +5:30
:slug: debvulns-needrestart
:tags: debvulns, cli, prometheus, debian, needrestart
:author: copyninja
:summary: Announcing release of debvulns with new features related to
          needrestart and kernel upgrade handling

After quite some time, I picked up ``debvulns`` again to implement what I
mentioned in my `previous post
<https://copyninja.in/blog/debvulns-exporter-newversion.html>`_: exporting reboot
required status and identifying services that need a restart.
Additionally, I wanted to ensure kernel vulnerabilities are not marked as
fixed unless the running kernel matches or exceeds the fixed version. Both
features are now live in version ``0.2.4`` (intermediate release ``0.2.3``
only included the needrestart functionality).

Why needrestart?
================

When you update packages on a system, security fixes do not take effect until
running processes actually use the updated binaries from disk. When an upgrade
occurs, existing processes continue executing the older, vulnerable code loaded
into memory, while only newly spawned instances pick up the fixed binary.
The ``needrestart`` package in Debian exists specifically for this purpose. It
can run in either report mode or action mode: report mode provides machine-readable
output, while action mode automatically restarts affected services.

Why integrate needrestart into debvulns?
----------------------------------------

``needrestart`` is a standalone utility that can run independently or hook into
``apt`` to run automatically after an upgrade. On desktop systems, it typically
runs interactively with a TUI prompt; on headless servers, it can be set to take
non-interactive default actions. In batch mode, it avoids restarting sensitive
system components (such as ``dbus``) to prevent destabilizing the session.

Integrating this into ``debvulns`` is not about replacing ``needrestart`` or
``debsecan``, but rather bringing vulnerability and remediation state into a single,
discoverable interface. ``debvulns`` aggregates these signals into a standard
format—in this case, Prometheus metrics and tags that can be visualized in
Grafana, monitored by alerting systems, or consumed downstream by automation tools.

Why Special Handling for Kernel?
================================

Just as user-space daemons require a restart after patching, a patched kernel
requires a system reboot into the new image before the vulnerability is truly
mitigated. Previously, ``debvulns`` only verified whether the patched package
version was installed on disk. If the deb package was installed, the CVE was
marked as resolved, which creates a false sense of security while running the old
kernel.

With this release, ``debvulns`` now checks the active running kernel release. A
kernel vulnerability is considered resolved only when the system is running the
fixed kernel version. Until a reboot happens, the system remains reported as
vulnerable.

The updated dashboard reflects these changes below. It will need further polish,
but key metrics and restart/reboot states are now visible:

.. image:: {static}/images/needrestart-debvulns.png
   :alt: debvulns Grafana dashboard with needrestart and kernel metrics

What Next?
==========

These were the two pending functional areas I wanted to cover, and no further major
features are planned for now. Next up will be reviewing the entire codebase to
identify potential optimizations and remove any unnecessary complexity. I haven't
done a full pass through the repository since the initial skeleton implementation,
so testing edge cases and cleaning up the internals will be the main focus.
This will likely take some time since I only hack on this during weekend downtime.
Until then, happy hacking, give the new release a try, and file an issue if you
run into bugs.
