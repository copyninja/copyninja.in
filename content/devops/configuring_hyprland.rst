Using Gemini CLI to Configure the Hyprland Window Manager
#########################################################

:date: 2026-03-15 12:35
:slug: agentic_hyprland_config
:tags: gemini cli, agentic workflows, hyprland
:author: copyninja
:status: draft
:summary: A guide on migrating to the Hyprland window manager using the Gemini
          CLI to automate configuration and reduce system bloat.

What led to this experiment? Well, for one, there was this:

.. raw:: html

   <blockquote class="twitter-tweet" data-dnt="true">
     <a href="[https://twitter.com/karpathy/status/2026731645169185220](https://twitter.com/karpathy/status/2026731645169185220)"></a>
   </blockquote>
   <script async src="[https://platform.twitter.com/widgets.js](https://platform.twitter.com/widgets.js)" charset="utf-8"></script>

Recently, I spoke with `Ritesh <https://researchut.com/>`_, who mentioned his
success using the Gemini CLI to debug an idle power drain issue on his laptop. I
wanted to experiment with this myself, and I had the perfect use case:
configuring the Hyprland Window Manager on my aging laptop.

The machine is nearly eight years old with 12GB of RAM (upgraded from the
original 4GB). I found that GNOME and KDE were becoming overkill, often leading
to system freezes when running multiple AI-powered IDEs like Antigravity and VS
Code with Co-pilot. Coincidentally, I noticed my Jio number had a "Google One
2TB" and "Google AI Premium" plan available to claim. I claimed it, and now here
I am, experimenting with the Gemini CLI.

Getting Started
===============

First, you need to install ``geminicli``. It is an open-source project, and
currently, the easiest way to install it is via the Node Package Manager (npm):

.. code:: sh

    npm install -g @google/gemini-cli

Next, we need to create a context for Gemini—a set of instructions for it to
follow throughout the project. This is managed via a ``GEMINI.md`` file. I went
to Google Gemini, explained my requirements, and asked it to generate one for
me.

**My requirements were:**

1. A minimalist but fully functional session, comparable to my existing GNOME
   setup.
2. Basic functionalities including wallpaper, screen locks, and a status bar
   with system icons.
3. Swapping Control and Caps Lock (a must for Emacs users).
4. Mandatory permission prompts for privileged operations; otherwise, it can
   work freely within a specified directory.
5. Persistent memory/artifacts for the session.
6. Permission to inspect my current session to understand the existing hardware
   and software configuration.

The goal was to reduce bloat and reclaim memory for heavy applications like
Antigravity and VS Code. Gemini provided the following ``GEMINI.md`` file:

.. code-block:: markdown

    # Role: Hyprland Configuration Specialist (Minimalist & High-Performance)

    You are a Linux Systems Engineer specializing in migrating users from heavy
    Desktop Environments to minimalist, tiling-based Wayland sessions on Debian.
    Your goal is to maximize available RAM for heavy applications while maintaining
    essential desktop features.

    ## 1. Environment & Persona
    - **Target OS:** Debian (Linux)
    - **Target WM:** Hyprland
    - **Hardware:** ThinkPad E470 (i5-7th Gen, 12GB RAM)
    - **User Profile:** Emacs user, prioritizes "anti-gravity" (zero bloat).
    - **Tone:** Technical, concise, and security-conscious.

    ## 2. Core Functional Requirements
    - **Status Bar:** `waybar` (with CPU, RAM, Network, and Battery icons).
    - **Wallpaper:** `swww` or `hyprpaper`.
    - **Screen Lock:** `hyprlock` + `hypridle`.
    - **Input Mapping:** Swap Control and Caps Lock (`kb_options = ctrl:nocaps`).

    ## 3. Operational Constraints
    - **Permission First:** Ask before using `sudo` or writing outside the work directory.
    - **Inspection:** Use `hyprctl`, `lsmod`, or `gsettings` for compatibility checks.
    - **Artifact Management:** Update `MEMORY.md` after every major step.

Gemini also recommended creating a ``MEMORY.md`` file to track progress.
Interestingly, Gemini remembered that I had previously shared ``dmidecode``
output, so it already knew my exact laptop specs. (Though it did include a note
about me being a "daily rice eater"—I assume it meant Linux 'ricing,' though I
actually use Debian Unstable, not Stable!).

The AI suggested starting with this prompt:

 Read MEMORY.md and GEMINI.md. Based on my hardware, give me a shell script to
 inspect my current GNOME environment so we can start replicating the session
 basics.

How Did It Go?
==============

I initialized a git repository for these files and instructed the Gemini CLI to
update ``GEMINI.md`` and commit changes after every major step so I could track
the progress.

The workflow looked like this:

1. **Inspection:** It created a `script
   <https://github.com/copyninja/hyprland-config/blob/main/inspect_gnome.sh>`_
   to extract my GNOME settings.
2. **Configuration:** Once I provided the output, it began configuring Hyprland.
3. **Utilities:** It generated an `installation script
   <https://github.com/copyninja/hyprland-config/blob/main/install_utils.sh>`_
   for all required Wayland utilities.
4. **Validation:** All changes were staged in a ``hypr-config-draft`` folder. I
   had Gemini verify them using ``hyprland --verify-config`` before moving them
   to ``~/.config/hypr``.

Most things worked immediately, but I hit a snag with the wallpaper. Even after
generating the config, ``hyprpaper`` failed to display anything. The AI got
stuck in a loop trying to debug it. I eventually spawned a second Gemini CLI
instance to review the code and logs.

The debug log showed: ``'DEBUG ]: Monitor eDP-1 has no target: no wp will be
created'``.
It turns out the configuration format was outdated. By feeding the `Hyprpaper
Wiki <https://wiki.hypr.land/Hypr-Ecosystem/hyprpaper/>`_ into the AI, it
finally corrected the config, and the wallpaper appeared.

After that, it successfully fixed an ``ssh-agent`` issue and configured a
clipboard manager with custom keybindings.

Learnings
=========

I have used window managers for a long time because my hardware was rarely
top-of-the-line. However, I had moved back to KDE/GNOME with the arrival of
Wayland because most of my preferred WMs were X11-based.

Manually configuring a window manager is a painful, time-consuming process
involving endless wiki-trawling and trial-and-error. What usually takes weeks
took only a few hours with the Gemini CLI.

AI isn't perfect—I still had to step in and guide it when it hit a wall—but the
efficiency gain is undeniable. If you're interested in the configuration or the
history of the session, you can find the repository `here
<https://github.com/copyninja/hyprland-config>`_.

I still have a few pending items in ``MEMORY.md``, but I'll tackle those next
time!
