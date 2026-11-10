Experiment: Debian Cloud/VM Images using mkosi
##############################################

:date: 2026-11-10 12:30 +5:30
:slug: debian-cloud-mkosi
:tags: debian, cloud, mkosi, vm images
:author: copyninja
:summary: Experiment to convert debian-cloud-images into mkosi based images with
          some modification

Debian's `Cloud Images <https://cloud.debian.org/images/cloud/>`_ for various
public cloud platform and VM images are based on a utility called
`debian-cloud-images <https://tracker.debian.org/pkg/debian-cloud-images>`_, a
inhouse utility developed and maintained by *Debian Cloud Team*.

What?
=====

The *debian-cloud-images* use FAI (Fully Automated Installation) for creating
custom VM images for public clouds like *GCP*, *Azure* and *AWS* along with
*genericcloud* class. I wanted to convert this into fully declarative mkosi
based image where entire image is described using INI format files with only few
scripts (postinst scripts) to do some final customization.

Why?
====

I've been working with various image customization format from long, in my
previous company I used to build customised industrial PC distribution based
purely on Debian using *multistrap* and bunch of shell scripts to hook together
raw image creation installation of bootloader etc. I've also did custom installs
using *debbootstrap* on my own laptop with *systemd-boot* as boot loader but
mostly hand built not automated. And once I came across the *mkosi* I'm just
impressed one utility do everything and churn out fully usable image with no
additional scripts for partition and image creation etc. You will still need few
scripts to finalise some of installatiion which goes via postinst but that is
very limited.

So I decided why not convert the *debian-cloud-images* to fully declarative
format of mkosi with a few changes. My requirement was to used

1. btrfs as the root file system. Because I like it one thing and other ability
   to create subvolumes and snapshots allowing atomic rollbacks when things go
   wrong.
2. systemd-boot integration, mainly because I feel its very clean and small like
   how bootloader should be and allows modern integration like unified kernel
   images uefi secureboot enabling as a bliss compared to the GRUB world.

Implementation
==============

Ofcourse the entire implementation was done mostly by agy with few debugging and
cleaning up things from my side. I pointed agy to debian-cloud-images local copy
and told it my requirements along with keeping original images intact and
ability to build it selectively for each cloud. The final generated `spec is
here
<https://github.com/copyninja/debian-cloud-mkosi/blob/main/spec/SPECIFICATION.md>`_.
Reposiory also has a `script <https://github.com/copyninja/debian-cloud-mkosi/blob/main/scripts/verify-image-parity.sh>`. which verifies functional integrity with original
image created by *debian-cloud-images*, except the root being btrfs nad use of
systemd-boot. Initial verification is done for all images of trixe and the
pairity report is published `here
<https://github.com/copyninja/debian-cloud-mkosi/blob/main/PARITY_REPORT.md>`_.

I've also booted *genericcloud* profile image and verified its bootability and
usefulness. This completes the phase 1 of implementation the reamining phases
which are captured in spec include

1. Phase 2: Immutable Images with dm-verity:

   Configure Verity=hash in systemd-repart 10-root.conf. Enable mkosi
   auto-signing for verity roothash and pass roothash into UKI command lines
   automatically.

2. Phase 3: Secure Boot & Signed UKIs:

    Integrate SecureBoot=yes with custom owner keys or ShimBootloader=signed
    using Microsoft Third-Party CA signed shim binaries.

3. Phase 4: System Extensions (systemd-sysext):

   Decouple cloud vendor agents (AWS CLI, Azure waagent, GCP guest agent) into
   separate .raw system extensions. Maintain a single unified, signed base OS
   image across all cloud platforms.

4. Phase 5: Btrfs Automated Rollbacks:

   Combine Btrfs @snapshots with systemd-boot snapshot boot entries for
   fail-safe OS upgrades.

One good thing about this setup is you can also simply use this image to setup
your laptop or server directly. For eg. in my `earlier post
<https://copyninja.in/blog/live_install_debian.html>`_ and following post I was
using live boot with debootstrap and then manually installing systemd-boot etc
to do a complete custom UEFI setup. But with mkosi I can simply spawn a image
with all required things locally (UKI signing) and all I would need is live boot
to write this image to root disk.

Conclussion
===========

This experiment is just exploring usefulness of mkosi, I've not discussed with
*Debian Cloud Team* on its adoption. Unless they are willing this will continue
as my pet project and of course I won't be able to provide pre-built images but
any one interested is free to clone and build the required images.

I will probably take some more time to come up with dm-verity based images
because I need to understand it first before playing with it. Till then happy
hacking!. Please report any issues and improvement suggestion on `Github <https://github.com/copyninja/debian-cloud-mkosi/issues/>`_.
