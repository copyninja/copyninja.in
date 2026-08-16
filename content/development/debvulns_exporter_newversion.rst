Releasing debvulns-exporter and debvulns cli 0.2.2
##################################################

:date: 2026-08-16 17:00 +5:30
:slug: debvulns-exporter-newversion
:tags: debvuns, cli, prometheus, debian
:author: copyninja
:summary: Announcing release debvulns 0.2.2 with enhancements and updated
          dashboard


I made another minor release but with some enhancements to handle non Debian
origin vulnerabilities and improving the data caching and sharing the same cache
between the debvulns cli as well as exporter. Additionally some improvements on
dashboard front as well. So here is what really changed

Handling vulnerabilities in Non Debian origin Packages
======================================================

During previous release I noticed that `grafana` package which is not really in
Debian and installed via upstream repository was shown as vulnerable with
multiple vulnerabiltiy. I got curious on why this is happening and found that
all CVE reported in dashboard are indeed present on
`security-tracker.debian.org` but with no fixed or any other description. So the
logic naturally thought no fix is available and hence it showed up as vulnerable
on dashboard.

How Did I solve this?
---------------------

There is a distributed vulnerability database maintained by Google for Open
Source project vulnerabilities called `osv.dev`. I looked up the vulnerabilities
in this dashboard for few shown in my dashboard and looked up generic
vulnerability information i.e. not tied to any distribution and there I could
find that this vulnerability is already fixed in the version I'm running. So
essentially what I needed really was differentiate the package from Debian and
not from Debian which is what origin field in apt mentions about.

Pitfall
-------

AI got it right and wrote code to differentiate the origin and non-origin based
on `apt_pkg.PackageRecords` `origin` field but then I noticed that a lot of
Debian packages are marked as non Debian origin. On closely checking I noticed
that if package is having a new version available then installed versions origin
is unset so had to work out a way to find that there is upgrade available and
use upgraded version to find the origin which was done by this `patch
<https://github.com/copyninja/debsecan-mcp/commit/cc72d477d0d0a6ff2ba5d2b70007bfc13fd50172>`_
. This solution was proudly crafted by me ;-) (reason I ran out of limits and
waiting for 6 hours for next reset).

Caching OSV Data
----------------

Initially AI wrote the code such that every time I run the exporter it will
re-download entire OSV data which was unnecessary. Also these information is not
quickly changing once its published so it also makes sense to store it and for
longer period that vulnerability and epss cache which has only 24h time on disk
before refresh. Now OSV cache for vulnerabilities will be present for 7 days
before triggering a fresh download.

All the cache expiry is configurable using command linie parameter.

Catch
-----

One catch of this feature is, I've not verified if every CVE published is
present on `security-tracker.debian.org` or not. In case of `grafana` which is
not packaged to Debian its present. So this feature will work with this
assumption that `security-tracker.debian.org` holds all CVE information
irrespective of if its in Debian or not. I will need to recheck on this and add
handling if that is not the case.


Unified Cache Folder for both CLI and Exporter
==============================================

Next thing I noticed was while AI was developing the code `debvulns` cli utility
was using `/var/cache/debvulns` where as the Prometheus exporter was using
`/var/cache/debvulns-exporter`. This is fine as long as any one utility is
installed on the system, but imagine some one has both installed and in this
case there is duplication of cache content and I felt I should avoid this
because core logic is same for both these utilities so why not have same cache
for both so there is no duplicate downloads saving a lot of time. 

Dashboard Changes
=================

During initial release of dashboard I noticed that in my test setup which
included my own laptop and a old Debian 12 and Debian 11 VM the vulnerability
count was very high that got me wondering is this unique list or some
vulnerabilties are just present on all 3 and got replicated. This is also some
normal question I get at work place while discussing on vulnerabilities,

- How many unique vulnerabilities are present?
- How many unique list of packages are affected?

So I thought why not codify this in the dashboard itself. So dashboard has been
redesigned to give this information along with list of packages affected. New
dashboard now looks like this, you can compare it with screenshot from my
previous post.

.. image:: {static}/images/new_debvulns_dashboard.png


What Next?
==========

I still have few things in my mind which I think I should implement to make
debvulns an complete vulnerability package for Debian which gives complete
picture of vulnerability status of system. These include

1. Kernel Vulnerability handling: Currently if fixed kernel is installed then
   vulnerabiliy is considered as fixed but in reality that is not the case.
   System is vulnerable as long as running kernel is still vulnerable
   irrespective of packae status on disk. So adding this ability is crucial.
2. Reboot required and service restarts: Just like kernel vulnerabiliy requires
   reboot to new kernel many package vulnerabilities fix requires running
   services using those binaries to be restarted to use fixed libraries /
   binaries. This is currently exposed by a package called `needrestart`. I
   think including this functionality inside debvulns makes sure we provide full
   package in one place which is easier to consume or dashboard for visibility.
3. Making this package available in Debian: With all these the package should be
   made available in Debian so people can simply install it and get the
   vulnerability status of their infra or machine. So this will be last part of
   puzzle I intend to do once above 2 are handling.

Till then happy hacking.
