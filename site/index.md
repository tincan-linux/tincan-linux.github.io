+------------------------------------------------------------------------------+
|  Tin Can Linux, a distro "made from scraps I found on the internet" (TM).    |
+------------------------------------------------------------------------------+

Tin Can Linux is an independent hobby distribution made with the goal of being
compact, understandable, hackable, and easy to maintain.


=== Goals
=========

Here are some of my goals for the Tin Can Linux project. These may change over
time as the project grows and develops:

  - Provide a working desktop environment with a minimal browser
  
  - Keep the system reasonably compact (fewer packages, less bloaty stuff)
  
  - Maintain a set of repositories that is as small as possible while achieving
    the aforementioned goals (to ensure that I can reasonably keep this alive)


=== Package Management
======================

Tin Can Linux uses the Arc package manager (also a creation of my own). It
functions somewhat similarly to the kiss package manager but is notably
different in several ways. Read more about it at [@/wiki/arc](/wiki/arc).


=== Filesystem
==============

The filesystem is inspired by [sta.li](https://sta.li). The main difference from most distros is
the choice to replace /usr with a symlink to /. The directories are as follows:

      /bin - executables
     /boot - boot files
      /dev - devices
      /etc - system config
     /home - all user directories (including root)
  /include - include \& header files
      /lib - static / dynamic libs
      /opt - yucky stuff (nothing yet!)
     /proc - proc files
      /run - mountpoint for runfs
     /sbin - symlink to /bin
    /share - man pages, other package files
      /sys - sys files
      /tmp - mountpoint for tmpfs
      /usr - symlink to /
      /var - services, package repos, installed packages, misc filesystems


See more about these choices at [@/wiki/filesystem](/wiki/filesystem).


=== Contributing
================

See the github. Different parts of the project have slightly different
contribution guidelines.


=== Inspiration
===============

Tin Can Linux was inspired by and made possible by the following amazing
projects (in no particular order):

  - Mussel (https://github.com/firasuke/mussel)
  - StaLi (https://sta.li)
  - Linux from Scratch (https://linuxfromscratch.org)
  - Kiss Linux (https://kisslinux.github.io)
