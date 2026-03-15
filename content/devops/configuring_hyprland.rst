Using Gemini CLI to configure hyprland Winow Manager
####################################################

:date: 2026-03-15 12:35
:slug: agentic_hyprland_config
:tags: gemini cli, agentic workflows, hyrpland
:author: copyninja
:status: draft
:summary: This post talks about getting the hyrpland window manager configured
          using Gemini CLI


What lead to this experiment?, well one is this

.. raw:: html

   <blockquote class="twitter-tweet" data-dnt="true">
     <a href="https://twitter.com/karpathy/status/2026731645169185220"></a>
   </blockquote>
   <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

And recently I spoke to `Ritesh <https://researchut.com/>`_ who was mentioning
of his experiment with Gemini CLI to debug a Idle Power drain issue on his
laptop. I also wanted to experiment this myself and I had a perfect use case!
Configuring Hyprland Window manager on my old laptop!. The laptop was almos 8
year old and having 12GB memory which was upgraded from original 4GB. The GNOME
and KDE was overkill and often lead to freeze when launching multiple AI powered
IDE like Antigravity and VS Code with co-pilot.

I also noticed that my Jio number had 18 months of Google 2T storage and Google
AI Pro plan available to be claimed. I claimed it and here I'm experimenting
with Gemini CLI.

How to get Started?
===================

First thing is to install geminicli which is a open source project but currently
only way to install it using node package manager (npm)

.. code:: sh

    npm install -g @google/gemini-cli


Next we need to create a context for Gemini like set of instructions on what it
needs to follow for the project. This is done using `GEMINI.md` file. So I went
to Google Gemini and told it my requirement and asked it to create one for me
;-).

   Can you help me in creating GEMINI.md whcih I want to use with gemini cli to
   help me configure hyprland window manager session.

   1. I need minimalist but fully working session like my existing session with gnome
   2. I want basic functionalities including wallpaper screenlocks and window
      bar indicating various icon
   3. I want to swap my control and caps session as I'm a emacs user
   4. I want it to ask for permission for privileged operations rest it can do
       in the work directory specified to it.
   5. I want it to save the memory/artifacts of this session
   6. It can inspect my existing session to understand existing configuration

   Add anything else which can help me use gemini cli affectively. Idea is to
   reduce bloat and efficiently use existing memory to applications like
   antigravity/vscode etc.

This provided me this raw GEMINI.md file

.. code-block:: markdown

    # Role: Hyprland Configuration Specialist (Minimalist & High-Performance)

    You are a Linux Systems Engineer specializing in migrating users from heavy
    Desktop Environments (like GNOME) to minimalist, tiling-based Wayland
    sessions on Debian. Your goal is to maximize available RAM for heavy
    applications like VS Code while maintaining essential desktop features.

    ## 1. Environment & Persona
    - **Target OS:** Debian (Linux)
    - **Target WM:** Hyprland
    - **Hardware Profile:** ThinkPad E470 (i5-7th Gen, 12GB RAM)
    - **User Profile:** Emacs user, prioritizes "anti-gravity" (zero bloat) and
      efficiency.
    - **Tone:** Technical, concise, and security-conscious.

    ## 2. Core Functional Requirements
    When generating configurations or scripts, prioritize these specific tools:
    - **Status Bar:** `waybar` (configured with icons for CPU, RAM, Network, and
      Battery).
    - **Wallpaper:** `swww` or `hyprpaper`.
    - **Screen Lock:** `hyprlock` + `hypridle`.
    - **Permissions/Privilege:** `polkit-kde-agent-1` or `lxqt-policykit` for
      GUI-based auth.
    - **Input Mapping:** Mandatory swap of Control and Caps Lock (`kb_options =
      ctrl:nocaps`).

    ## 3. Operational Constraints & Workflow
    - **Permission First:** You MUST ask for permission before suggesting `sudo`
      commands or writing files outside the specified work directory.
    - **Existing Session Inspection:** Before recommending new configs, ask the
      user to provide outputs of `hyprctl`, `lsmod`, or `gsettings` to ensure
      hardware/driver compatibility.
    - **Artifact Management:** After every major setup step, summarize the
      changes into a `MEMORY.md` file in the work directory. This serves as the
      session's "long-term memory."
    - **Directory Focus:** All generated scripts and config drafts should stay
      within the user-defined work directory until the user gives the "deploy"
      command.

    ## 4. Optimization Directives
    - Keep Hyprland animations and blur at a minimum to save GPU cycles on the
      i5-7200U.
    - Suggest Wayland-native alternatives for everything (avoid Xwayland where
      possible).
    - Focus on `libinput` optimizations for the ThinkPad TrackPoint and
      Touchpad.

    ## 5. Session Memory Template
    If `MEMORY.md` exists, read it first to understand current progress. If not,
    create it with:
    - **Status:** (e.g., Waybar configured, Lockscreen pending)
    - **Key Artifacts:** (Paths to generated scripts/configs)
    - **Hardware Notes:** (Drivers or specific ThinkPad tweaks applied)

It also recommended creation of `MEMORY.md` and I said fine whatever is required
and it spit out this content for `MEMORY.md`

.. code-block:: markdown

    # SESSION MEMORY: Hyprland Migration

    ## 1. System Context (Immutable)
    - **Hardware:** Lenovo ThinkPad E470 (Intel i5-7200U, 12GB RAM, Integrated
      Graphics).
    - **Distro:** Debian (Linux).
    - **Existing DE:** GNOME (to be used as a reference for functionality).
    - **Keyboard Layout:** Emacs-optimized (Caps Lock -> Control mapping
      required).
    - **Goal:** "Anti-gravity" setup (Zero-bloat, maximize RAM for VS Code/Heavy
      Apps).

    ## 2. Progress Status
    - [x] Initial GEMINI.md system prompt defined.
    - [ ] Base `hyprland.conf` generated.
    - [ ] Input rules (Caps/Ctrl swap) verified.
    - [ ] Core utilities (Waybar, Hyprlock, SWWW) configured.
    - [ ] Polkit agent for privileged operations integrated.

    ## 3. Configuration Artifacts
    - **Work Directory:** `./hypr-config-draft/`
    - **Draft Configs:** `None yet.`
    - **Hardware Tweaks:** - Priority: Libinput optimization for ThinkPad
      TrackPoint and Touchpad.
    - Priority: Disable heavy blur/animations to protect i5-7200U performance.

    ## 4. Pending Inspections
    - [ ] Run `gsettings list-recursively` to identify GNOME background services
      to replicate.
    - [ ] Check `lsmod | grep -i video` to confirm driver status for Wayland.
    - [ ] Capture existing display scaling (100% vs 125%) from GNOME session.

    ## 5. Session Notes
    - *Note:* User is a daily rice eater and prefers stability; ensure
      Debian-stable compatible package names are used in all scripts.

This file looks like initial todo list sort of thing for AI to get started with
and interestingly Gemini remembered my inputing `dmidecode` output earlier so it
knows exact specs of my laptop. By the way I'm not sure what was that note line
intended as ;-) (daily rice eater eh? and no I use Debian unstable)

It asked me to start with following prompt

  "Read MEMORY.md and GEMINI.md. Based on my hardware, give me a shell script to
  inspect my current GNOME environment so we can start replicating the session
  basics."

For keeping track of work I initialized a git repository where I placed the
above files and also asked Gemini cli itself to update Gemini.md to make sure it
commits after each changes so I've track of what happened in the repo.

How did It go?
==============

I started with recommended prompt and waited it for completion and since I had
asked to take permission for any privileged command I had to keep inspecting and
allowing it.

1. It initially it created a `script
   <https://github.com/copyninja/hyprland-config/blob/main/inspect_gnome.sh>`_
   to get existing gnome settings and configurations.
2. Once I provided it with generated output file it went ahead for configuring
   hyprland.
3. I had to ask it to make sure that all required utilities are installed, and
   it promptly generated another script for me to `execute
   <https://github.com/copyninja/hyprland-config/blob/main/install_utils.sh>`_.
4. One good thing Gemini did was it was doing all changes in `hypr-config-draft`
   and I finally asked it to verify changes using `hyprland --verify-configg`.
   Once everything is validated those are moved to `~/.config/hypr` folder.

Once it completed all changes I logged out and logged in and most of things were
working fine. But wallpaper was not appearing so I asked it to configure
`hyprpaper` it did ask me for Wallpaper and even after generating config and
re-logging in wallpaper was not appearing at all. It tried to debug but it went
in endless loop of doing things with no result. This is something which I
commonly notice with the AI. So I did a bit of research and also cloend the code
and spawned another Gemini CLI and asked it to go through the code to figure out
what was wrong and why wallpaper was not created. This was the error message
which I was getting with original configuration it created

.. code-block:: sh

    'DEBUG ]: Monitor eDP-1 has no target: no wp will be created'

After going through code it pointed out that configuration format is old but
couldn't rectify it. I then go `wiki page of hyprpaper
<https://wiki.hypr.land/Hypr-Ecosystem/hyprpaper/>`_ and asked it to fix the
configuration and finally it did that and wallpaper appeared.

Next I asked it to fix the ssh-agent issue I was facing whichi it properly fixed
and post that I asked it to configure clipboard and it did it promptly including
configuring the keybindings for using it!.

Learnings
=========

I've been user of the window manager for long time as I always had not very
powerful laptops. I had to switch to KDE/Gnome post arrival of Wayland as most
window manager I used were only X based. The task of configuring a window
manager to your need is a painful process. You need go through various sources
Wiki and trial and error to finally achieve a perfect configuration. This would
take weeks to months based on your free time. So Gemini CLI is definitely a Win
in this situation. I had a unknown window manager setup in few hours to my
perfect taste!. AI is not perfect as seen by some issue I faced and I had to
debug a bit and guide it to fix it but that is worth the effort. So all in all
its really useful to get things done very quickly.

The entire history and project used for this purpose is `here
<https://github.com/copyninja/hyprland-config>`_ if any one is interested at
looking into it. I've some final stuffs pending to configure as per `MEMORY.md`
I will try to finish those till next time!.
