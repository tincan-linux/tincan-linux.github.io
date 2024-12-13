+------------------------------------------------------------------------------+
|  Tin Can Linux, a tiny distribution scrapped together with hidden gems from  |
|  the depths of the Linux community.                                          |
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


=== Release Model
=================

Tin Can Linux follows a rolling release model. However, "versions" will be
provided in the form on tags on the repos for stable combinations of packages or
before making major changes to the repos.


=== Filesystem
==============

The filesystem is inspired by [sta.li](https://sta.li). The main difference from most distros is
the choice to replace /usr with a symlink to /. See more about these choices
at [@/wiki/filesystem](/wiki/filesystem).


=== Contributing
================

See the github. Different parts of the project have slightly different
contribution guidelines. In general, issues are welcome everywhere, and pull
requests are welcome everywhere except the official package repositories.


=== Inspiration
===============

Tin Can Linux was inspired by and made possible by the following amazing
projects (in no particular order):

  - Mussel ([https://github.com/firasuke/mussel])
  - Linux from Scratch ([https://linuxfromscratch.org])
  - Glaucus Linux ([https://glaucuslinux.org])
  - Kiss Linux ([https://kisslinux.github.io])
  - StaLi ([https://sta.li])
  - Oasis Linux ([https://github.com/oasislinux])
