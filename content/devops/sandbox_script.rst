Hardening the Unpacakgeable: A systemd-run Sandbox for Third-Party Binaries
###########################################################################

:date: 2026-04-12 12:53
:slug: safe-run-binary-sandbox
:tags: sandbox, debian, security, systemd, npm
:author: copyninja
:summary: A technical deep-dive into sandboxing untrusted npm and Python
          binaries using systemd-run and Linux namespaces.

The Shift in Software Consumption
=================================

Historically, I have been a "distribution-first" user. Sticking to tools
packaged within the Debian archives provides a layer of trust; maintainers
validate licenses, audit code, and ensure the entire dependency chain is
verified. However, the rapid pace of development in the Generative AI
space—specifically with new tools like Gemini-CLI—has made this traditional
approach difficult to sustain.

Many modern CLI tools are built within the **npm** or **Python** ecosystems. For
a distribution packager, these are a nightmare; packaging a single tool often
requires packaging a massive, shifting dependency chain. Consequently, I found
myself forced to use third-party binaries, bypassing the safety of the Debian
archive.

The Supply Chain Risk
=====================

Recent supply chain attacks affecting widely used packages like ``axios`` and
``LiteLLM`` have made it clear: running unvetted binaries on a personal system
is a significant risk. These scripts often have full access to your ``$HOME``
directory, SSH keys, and the system D-Bus.

After discussing these concerns with a colleague, I was inspired by his
approach—using a Flatpak-style sandbox for even basic applications like Google
Chrome. I decided to build a generalized version of this using **OpenCode** and
**Qwen 3.6 Fast** (which was available for free use at the time) to create a
robust, transient sandbox utility. 

The Solution: safe-run-binary
=============================

My script, `safe-run-binary
<https://github.com/copyninja/dotfiles/blob/master/bin/safe-run-binary>`_,
leverages ``systemd-run`` to execute binaries within an isolated scope. It
implements strict filesystem masking and resource control to ensure that even if
a dependency is compromised, the "blast radius" is contained.

Key Technical Features
======================

**1. Virtualized Home Directory (tmpfs)**
   Instead of exposing my real home directory, the script mounts a ``tmpfs``
   over ``$HOME``. It then selectively creates and bind-mounts only the
   necessary subdirectories (like ``.cache`` or ``.config``) into a virtual
   structure. This prevents the application from ever "seeing" sensitive files
   like ``~/.ssh`` or ``~/.gnupg``.

**2. D-Bus Isolation via xdg-dbus-proxy**
   For GUI applications, providing raw access to the D-Bus is a security hole.
   The script uses ``xdg-dbus-proxy`` to sit between the application and the
   system bus. By using the ``--filter`` and ``--talk=org.freedesktop.portal.*``
   flags, the app can only communicate with necessary portals (like the file
   picker) rather than sniffing the entire bus.

**3. Linux Namespace Restrictions**
   The sandbox utilizes several ``systemd`` execution properties to harden the
   process:
   
   * ``RestrictNamespaces=yes``: For CLI tools, this prevents the app from
     creating its own nested namespaces.
   * ``PrivateTmp=yes``: Ensures a private ``/tmp`` space that isn't shared with
     the host.
   * ``NoNewPrivileges=yes``: Prevents the binary from gaining elevated
     permissions through SUID/SGID bits.

**4. GPU and Audio Passthrough**
   The script intelligently detects and binds Wayland, PipeWire, and NVIDIA/DRI
   device nodes. This allows browsers like Firefox to run with full hardware
   acceleration and audio support while remaining locked out of the rest of the
   filesystem.

Usage
=====

To run a CLI tool like Gemini-CLI with access only to a specific directory:

.. code-block:: bash

   safe-run-binary -b ~/.gemini-config -- npx @google/gemini-cli

For a GUI application like Firefox:

.. code-block:: bash

   safe-run-binary --gui -b ~/.mozilla -b ~/.cache/mozilla -b ~/Downloads -- firefox

Conclusion
==========

While it is not always possible to escape the need for third-party software, it
is possible to control the environment in which it operates. By leveraging
native Linux primitives like ``systemd`` and namespaces, high-grade isolation is
achievable.

PS: *If you spot any issues or have suggestions for improving the script, feel free
to raise a PR on the* `repo <https://github.com/copyninja/dotfiles/>`_.
