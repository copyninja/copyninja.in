Releasing debvulns with needrestart and kernel vulnerability handling
#####################################################################

:date: 2026-11-01 18:03 +5:30
:slug: debvulns-needrestart
:tags: debvulns, cli, prometheus, debian, needrestart
:author: copyninja
:summary: Announcing release of debvulns with new features related to
          needrestart and kernel upgrade handling

After quite some time finally I again picked up debvulns to implement what I
mentioned in my `previous post
<https://copyninja.in/blog/debvulns-exporter-newversion.html>`_, ability to
export reboot required status and services needing restart information.
Additionally I wanted to make sure kernel vulnerabilities did not get marked as
fixed unless the kernel running is not same version as the fixed version. Both
these features are now up and running as part version `0.2.4` (intermediate
version `0.2.3`) only has needrestart related feature.

Why needrestart?
================

When we apply update to a system its not going to take effect untill all running
services are not using the fixed version of binary on disk. When upgrade happens
already running binaries will still run affected version of program from the RAM
while new launches only pick the updated version. `needrestart` application in
Debian is for this purpose only it can be configured to either report or action
mode. In report mode it outputs machine consumable text output and in action
mode we can make it restart all affected services.

Why not reuse needrestart?
--------------------------

`needrestart` is a stand alone binary which can be used independently or hooked
into `apt` system to be invoked at end of upgrade. In regular laptop it will run
in interactive mode giving users TUI to select what he needs to do and on server
we can just let it take default actions which is deemed correct. In automated
mode it will not touch services which should not be directly restarted like dbus
and other services to not cause issue to running system.

`debvulns` implementing this functionality just to make sure every security
related alternative are in one place for vulnerability management and provide
output in a standard format, in this case prometheus tags which can be
dashboarded or consumed by other services or AI to act on it. In no way
`debvulns` is taking over `needrestart` or `debsecan` its just providing single
discoverable interface for vulnerability management.


Why Special Handling for Kernel?
================================

Just like running services needs restart after upgrade to make sure fixed
version is in place, a kernel upgrade requires reboot into fixed kernel to say
system is no longer vulnerable. But earlier `debvulns` was only considering if
fixed version is installed or not. If fixed version was in the vulnerability was
marked fixed. This will give wrong signal about system security status. With
this fix now only if fixed kernel is running then only its marked as fixed else
till reboot system will shown as vulnerable.

The updated dashboard looks like below it will need some more refining but
whatever needs to be highlighted is captured as of now.

.. image:: {static}/images/needrestart-debvulns.png 

What Next?
==========

These are two points which I wanted to implement and as of now I don't see any
more feature which needs to be implemened. So next up will be reading the code
and seeing if any fix or simplication is needed. I have not completely read the
code other than initial skeleton code. So going through entire thing making sure
there is no unwanted complexity and making sure every corner cases tested will
be next thing. This last thing will take some time given I only check this on my
free time over weekend. Untill then happy hacking and do use the tool and report
any issues.
