Experiment: Debian Cloud/VM Images using mkosi
##############################################

:date: 2026-11-10 12:30 +05:30
:slug: debian-cloud-mkosi
:tags: debian, cloud, mkosi, vm images
:author: copyninja
:summary: Experiment to convert debian-cloud-images into declarative, mkosi-based images with Btrfs and systemd-boot.

Debian's `Cloud Images <https://cloud.debian.org/images/cloud/>`_ for various
public cloud platforms and VM environments are built using a tool called
`debian-cloud-images <https://tracker.debian.org/pkg/debian-cloud-images>`_,
an in-house utility developed and maintained by the *Debian Cloud Team*.

What?
=====

The *debian-cloud-images* tooling uses FAI (Fully Automatic Installation) to create
custom VM images for public clouds like *GCP*, *Azure*, and *AWS*, along with the
*genericcloud* class. I wanted to convert this into a fully declarative, mkosi-based
image build where the entire image is described using INI-format configuration files,
with only a few post-installation scripts to handle final customizations.

Why?
====

I've been working with various image customization formats for a long time. In my
previous company, I used to build customized industrial PC distributions based purely
on Debian using *multistrap* and a bunch of shell scripts to stitch together raw image
creation, bootloader installation, etc. I've also done custom installs using
*debootstrap* on my own laptop with *systemd-boot* as the bootloader, but that was
mostly hand-crafted and not automated. Once I came across *mkosi*, I was impressed by
how a single utility can do everything and churn out a fully usable image without extra
glue scripts for partitioning and image creation. You still need a few scripts to
finalize some parts of the installation (via postinst hooks), but those are very
minimal.

So I decided: why not convert *debian-cloud-images* into mkosi's declarative format
with a few architectural changes? My requirements were:

1. **Btrfs as the root filesystem**: Partly because I like it, but primarily for the
   ability to use transparent compression (`zstd:1`), subvolumes, and snapshots for
   atomic rollbacks when things go wrong.
2. **systemd-boot and Unified Kernel Images (UKI)**: Primarily because *systemd-boot*
   is clean and minimal—exactly what a modern bootloader should be. Combining it with
   UKIs and UEFI Secure Boot is bliss compared to the complexity of the GRUB2 world.

   **Note:** *This intentionally drops legacy BIOS (MBR/CSM) partitioning support in
   favor of pure UEFI. Modern cloud platforms (AWS Nitro, Azure Gen2, GCP) support
   UEFI natively, and dropping BIOS allows shedding significant legacy bootloader
   baggage—though it does make these images unsuitable for older, non-UEFI hypervisors.*

Implementation
==============

Of course, the initial heavy lifting was done mostly using Antigravity (*agy*), along with
debugging, tuning, and cleanup from my side. I pointed *agy* to a local copy of
*debian-cloud-images* and laid out my requirements: preserve parity with the original
packages and configurations, while enabling selective builds for each cloud via modular
profiles. The resulting `specification is documented here
<https://github.com/copyninja/debian-cloud-mkosi/blob/main/spec/SPECIFICATION.md>`_.

The repository also includes a `verification script
<https://github.com/copyninja/debian-cloud-mkosi/blob/main/scripts/verify-image-parity.sh>`_
that audits functional parity against official images produced by *debian-cloud-images*
(verifying packages, APT deb822 sources, and systemd services, while accounting for the
switch to Btrfs and systemd-boot). Initial verification was performed for all Trixie
profiles, and the parity report is published `here
<https://github.com/copyninja/debian-cloud-mkosi/blob/main/PARITY_REPORT.md>`_.

I've also booted the *genericcloud* profile image under UEFI in a local VM to verify
live bootability, `bootctl status`, network connectivity, and APT repository setup.
This completes Phase 1 of the implementation. The remaining phases outlined in the
specification include:

1. **Phase 2: Immutable Images with dm-verity**:
   Configure `Verity=hash` in systemd-repart's `10-root.conf`. Enable mkosi auto-signing
   for the verity root hash and pass the root hash into UKI kernel command lines
   automatically.

2. **Phase 3: Secure Boot & Signed UKIs**:
   Integrate `SecureBoot=yes` with custom owner keys or `ShimBootloader=signed` using
   Microsoft Third-Party CA signed shim binaries.

3. **Phase 4: System Extensions (systemd-sysext)**:
   Decouple cloud vendor agents (AWS CLI, Azure waagent, GCP guest agent) into separate
   `.raw` system extensions. Maintain a single unified, signed base OS image across all
   cloud platforms.

4. **Phase 5: Btrfs Automated Rollbacks**:
   Combine Btrfs `@snapshots` with systemd-boot snapshot boot entries for fail-safe OS
   upgrades.

Another advantage of this setup is that the same tooling can build images for physical
laptops and bare-metal servers. For example, in an `earlier post
<https://copyninja.in/blog/live_install_debian.html>`_, I walked through live-booting
with *debootstrap* and manually configuring *systemd-boot* for a custom UEFI
installation. With *mkosi*, I can simply generate a customized image locally with UKIs
ready to go, live-boot the target machine, and write the image directly to the root disk.

Conclusion
==========

This experiment is an exploration of the power of *mkosi*; I haven't discussed adoption
with the *Debian Cloud Team* yet. For now, this will continue as my pet project. While
I won't be hosting pre-built binary images, anyone interested is welcome to clone the
repository and build the images locally.

I will probably take some more time before tackling dm-verity images, as I want to
understand the nuances thoroughly before experimenting with them. Until then, happy
hacking! Please report any issues or suggestions on `GitHub
<https://github.com/copyninja/debian-cloud-mkosi/issues>`_.
