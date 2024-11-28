+------------------------------------------------------------------------------+
|  About the Tin Can Linux filesystem.                                         |
+------------------------------------------------------------------------------+

The Tin Can Linux filesystem intentionally deviates from standard Linux
filesystem layouts in an effort to be more organized.


=== Contents
============

[[010](#010)] Overview

[[020](#020)] Why?

[[030](#030)] To Do


=== Overview $[010]
==================

The filesystem is generally organized as follows:

--------------------------------------------------------------------------------

        /bin - executables
       /boot - boot files
        /etc - system config
       /home - all user directories (including root)
    /include - include \& header files
        /lib - static / dynamic libs
        /run - mountpoint for runfs
      /share - man pages, other package files
        /tmp - mountpoint for tmpfs
        /var - program data and misc filesystems


  With the following as compatibility symlinks:

      /sbin -> /bin
       /usr -> /


  Assuming some standard Linux filesystems:

        /dev - devices
       /proc - proc files
        /sys - sys files

--------------------------------------------------------------------------------


=== Why? $[020]
==============

The standard filesystem on most distros is complicated and not easy to
understand. This filesystem used by Tin Can Linux aims to simplify it by
condensing things and eliminating uncesessary parts.


=== To Do $[030]
===============

These are further changes that I would eventually like to include but am
currently unsure of how to go about / have not yet gotten around to working on.
Any suggestions would be much appreciated: [open an issue](https://github.com/tincan-linux/tincan/issues) or [make a discussion](https://github.com/tincan-linux/tincan/discussions).

  - Eliminate /run and use /var/run instead
