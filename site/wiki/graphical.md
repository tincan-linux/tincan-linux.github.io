+------------------------------------------------------------------------------+
|  Installing a graphical system.                                              |
+------------------------------------------------------------------------------+

This page provides more elaboration on installing a graphical environment on Tin
Can Linux.


=== Contents
============

[[010](#010)] Wayland
  [[011](#011)] Installation
  [[012](#012)] Seatd
  [[013](#013)] XDG_RUNTIME_DIR

[[020](#020)] Xorg

[[030](#030)] Fonts


=== Wayland $[010]
=================

Wayland is the primary graphical system for Tin Can Linux.


=== Installation $[011]

Currently, compositors based on wlroots are supported. Install wlroots (which
pulls in all other necessary packages as dependencies):

--------------------------------------------------------------------------------

  $ moss b wlroots

--------------------------------------------------------------------------------


=== Seatd $[012]

In order to start most compositors, you will need to enable the 'seatd' service
and add your user to the 'video' group:

--------------------------------------------------------------------------------

  Enable the seatd service
  $ ssu rc enable seatd

  Check that it's enabled
  $ rc list

  Add your user to the video group (replace xxxxx with your username)
  $ ssu adduser xxxxx video

--------------------------------------------------------------------------------


=== XDG_RUNTIME_DIR $[013]

This environment variable must be set in order to start a Wayland session. To
properly set it, add these lines to /etc/profile:

--------------------------------------------------------------------------------

if test -z "${XDG_RUNTIME_DIR}"; then
    export XDG_RUNTIME_DIR=/tmp/$(id -u)-runtime-dir
    if ! test -d "${XDG_RUNTIME_DIR}"; then
        mkdir "${XDG_RUNTIME_DIR}"
        chmod 0700 "${XDG_RUNTIME_DIR}"
    fi
fi

--------------------------------------------------------------------------------


=== Xorg $[020]
==============

Xorg was the primary display server until 11/30/2024 but may no longer be
maintained regularly because it's more difficult for me to keep up with the
updates. The process is similar to Wayland, minus seatd.

Instructions will be here. Soon (TM).


=== Fonts $[030]
===============

Most wayland packages, such as compositors, terminal emulators, etc will require
at least one font to be installed in order to work properly.

First, install fontconfig:

--------------------------------------------------------------------------------

  $ moss b fontconfig

--------------------------------------------------------------------------------


Next, find a font of your choice, preferrably one that's monospaced. Install it
to either /usr/share/fonts/ (global install) or ~/.local/share/fonts/ (user
install) and rebuild the font cache:

--------------------------------------------------------------------------------

  $ ssu mkdir -p /usr/share/fonts && cd /usr/share/fonts
  $ ssu curl -fLO <link to font.ttf>
  $ fc-cache

--------------------------------------------------------------------------------


Here are some interesting fonts I've used before, if you need some inspiration:
  - https://github.com/oppiliappan/scientifica
  - https://github.com/molarmanful/kirsch
