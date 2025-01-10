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


=== Wayland $[010]
=================

Wayland is the officially supported graphical system for Tin Can Linux.


=== Installation $[011]

Currently, compositors based on wlroots are supported. Install wlroots (which
pulls in all other necessary packages as dependencies):

--------------------------------------------------------------------------------

  $ arc b wlroots

--------------------------------------------------------------------------------


=== Seatd $[012]

In order to start most compositors, you will need to enable the 'seatd' service
and add your user to the 'video' group:

--------------------------------------------------------------------------------

  # Enable the seatd service
  $ ssu rc enable seatd

  # Check that it's enabled
  $ rc list

  # Add your user to the video group (replace xxxxx with your username)
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

Xorg was supported until 11/30/2024 but will no longer be officially maintained.
These instructions are here in case you want to use X (or in the event that
someone offers to unofficially maintain repo-xorg).

... I'm working on it. They will be here. Soon (TM).
