+------------------------------------------------------------------------------+
|  Tin Can Linux repositories.                                                 |
+------------------------------------------------------------------------------+

This page provides some details about the Tin Can Linux package repositories.


=== Contents
============

[[010](#010)] Official Repositories
  [[011](#011)] Core
  [[012](#012)] Extra
  [[013](#013)] Xorg
  [[014](#014)] Contributing to Official Repositories

[[020](#020)] Community Repository

[[030](#030)] Repository Structure


=== Official Repositories $[010]
===============================

There are currently three official repositories for Tin Can Linux:

  - [$/tincan-linux/repo-core](https://github.com/tincan-linux/repo-core): packages that any complete installation must have
  - [$/tincan-linux/repo-extra](https://github.com/tincan-linux/repo-extra): packages that are not strictly required to boot
  - [$/tincan-linux/repo-xorg](https://github.com/tincan-linux/repo-xorg): packages that are needed for a graphical environment


=== Core $[011]

This repository is reserved for packages that must be present on any
installation of Tin Can Linux. These include the package manager, toolchain, and
other essential software.


=== Extra $[012]

This repository contains everything else. These are packages that may be useful
or are needed as dependencies, but are not strictly required to boot.


=== Xorg $[013]

This repository contains any software related to the Xorg graphical system. This
repo will soon be unmaintained and dropped in favor of Wayland since it requires
fewer packages and will be easier for me to maintain in the long term.


=== Contributing to Official Repositories $[014]

Since I may eventually enable SSH signature verification for the official
repositories, I will not merge pull requests on the official repositories (core,
extra, and xorg). However, issues are always welcome (and encouraged!) if you
would like to suggest a package for inclusion in the official repositories.

Pull requests are welcome on all other parts of the project.


=== Community Repository $[020]
==============================

There are currently plans for a community repository, if there is enough
interest in the project. If there is a package that you would like to maintain
for Tin Can Linux, [start a discussion](https://github.com/tincan-linux/tincan/discussions) and I will be be happy to create a
community repository to include it in.


=== Repository Structure $[030]
==============================

Each one of the official repositories is simply a git repository where each
package is represented by its own directory. This closely mirrors the layout of
the [Glaucus Linux repositories](https://github.com/glaucuslinux/cerata). Further, each package is a directory with two
files: package.toml, which defines key package metadata, and build, which
provides instructions to build the package. More information about package
structure can be found at [@/wiki/arc](/wiki/arc#020).
