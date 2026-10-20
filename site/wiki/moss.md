+------------------------------------------------------------------------------+
|  Moss, a tiny Linux Package Manager.                                         |
+------------------------------------------------------------------------------+

Moss (formerly arc) is a tiny, lightning-fast package manager written by me
(AVS Origami). It is the package manager used by Tin Can Linux.

Repository: [$/avs-origami/moss](https://github.com/avs-origami/moss)


=== Contents
============

[[010](#010)] Usage
  [[011](#011)] Resolving conflicts

[[020](#020)] Package format
  [[021](#021)] Package.toml
  [[022](#022)] Build

[[030](#030)] Configuration
  [[031](#031)] Repo search path
  [[032](#032)] Other config keys

[[100](#100)] Appendix
  [[110](#110)] About $ARC_PATH


=== Usage $[010]
===============

To use moss, invoke it along with one of the available actions (or run without
arguments to print out this help message):

--------------------------------------------------------------------------------

  $ moss

  /\\/\\   ___  ___ ___ 
 /    \\ / _ \\/ __/ __|
/ /\\/\\ \\ (_) \\__ \\__ \\
\\/    \\/\\___/|___/___/

  Usage: moss [s/v/y][b/c/d/f/h/i/l/n/p/r/s/u/v] [pkg]...
    -> b / build     Build packages
    -> c / checksum  Generate checksums
    -> d / download  Download sources
    -> f / find      Fuzzy search for a package
    -> h / help      Print this help
    -> i / install   Install built packages
    -> l / list      List installed packages
    -> n / new       Create a blank package
    -> p / purge     Purge the package cache ($HOME/.cache/moss)
    -> r / remove    Remove packages
    -> s / sync      Sync remote repositories
    -> u / upgrade   Upgrade all packages
    -> v / version   Print version
  Flags:
    -> s  Sync remote repositories
    -> v  Enable verbose builds
    -> y  Skip confirmation prompts

  Created by AVS Origami

--------------------------------------------------------------------------------


Notes on usage:

  - Each action can be invoked by typing out the whole action or by using the
    single letter alias (e.g. 'moss build gcc' is equivalent to 'moss b gcc').

  - For actions that involve operating on packages, all arguments following the
    action are treated as packages to be operated on.

  - If the alias is used it can be combined with one or more flags to set
    one-time options.


Some usage examples:

--------------------------------------------------------------------------------

  Installs the 'gcc' package.

  $ moss b gcc

  Installs the 'gcc' and 'zlib' packages, and prints all output from the build
  script to the terminal (output will still be stored to the log files as well).

  $ moss vb gcc zlib

  Sync the local repositories with upstream and perform a full system upgrade,
  automatically bypassing all confirmation prompts.

  $ moss syu

--------------------------------------------------------------------------------


=== Resolving conflicts $[011]

Sometimes, a file will be provided by multiple packages (for example, the
program 'clear' is provided by both 'busybox' and 'ncurses'). In this case the
user must manually select which package should provide the file.

If a conflict is detected, moss will provide a [Y/n] prompt asking whether to
[Y] use the new provider or [n] keep the current provider. Any answer not
starting with 'n' or 'N' will be treated as 'Y'.

The package manager will update the manifests of installed packages as necessary
to accomodate changes, but the choice of provider is not otherwise remembered.
As such any time either package is updated the user will need to explicitly
state which provider to use.


=== Package format $[020]
========================

Moss packages consist of two main files: 'package.toml' and 'build'. At a high
level, these files serve the following functions:

  - The file 'package.toml' contains all package metadata (version, maintainer,
    and information about sources) as well as dependency requirements.

  - The file 'build' is an executable shell script that provides instructions
    to build the package and install it to a temporary destdir.


To illustrate this better, this is the directory structure of the 'libelf'
package (which I will use as an example throughout this section):

--------------------------------------------------------------------------------

  $ tree /var/repo/core/libelf

  libelf
  |-- build
  |-- files
  |   `-- musl.patch
  `-- package.toml

--------------------------------------------------------------------------------

Moss provides functionality to create a new package following this template;
just run 'moss new [name]' and it will create a new package called '[name]' in
the current directory containing a 'package.toml' and 'build'. These files must
then be edited according to the software being packaged.


=== Package.toml $[021]

This file provides important information about a package. It is split into three
distinct sections: a [meta] section, a [deps] section, and a [mkdeps] section.
As an example, here is the 'package.toml' for the 'libelf' package:

--------------------------------------------------------------------------------

  $ cat /var/repo/core/libelf/package.toml

  [meta]
  version = "0.191"
  maintainer = "AVS Origami <avs.origami@gmail.com>"

  sources = [
      "https://sourceware.org/elfutils/ftp/0.191/elfutils-0.191.tar.bz2",
      "files/musl.patch",
  ]

  checksums = [
      "e9e3f567ab63784d968f708c08ea5a09dd5fae2f0846d0d43a2ebc8b542c15b2",
      "0b92f76da7cc952bcb8490b80320449d8cc0905ca2ba8c7c29052eae99e631ec",
  ]

  [deps]
  zlib = "*"

  [mkdeps]
  pkgconf = "*"

--------------------------------------------------------------------------------


And an explanation of each of these fields:

  - [meta] provides basic package metadata. This includes:

      - version: the current version of the package.

      - maintainer: the package maintainer (if you're creating a package, this
        is you).

      - sources: a list of sources to download to the build directory. These
        files may be from the internet (typically a source tarball) or specified
        locally (patches, service files, etc). Some notes about sources:

          - Tarballs (*.tar.*) are automatically extracted to the build
            directory ($HOME/.cache/moss/build/[pkg]/src). This can be
            overridden by prefixing the url with 'tar+', in which case the
            tarball will be copied as-is to the build directory.

          - When a tarball is extracted, the top level directory is stripped
            away. This means that the sources are directly in the build
            directory (and not behind another directory like elfutils-0.191/).

      - checksums: a list of blake3 checksums for each source:

          - These correspond to the order in which the sources are provided (so
            here, 'e9e3f567...' is the checksum for 'elfutils-0.191.tar.bz2' and
            '0b92f76d...' is the checksum for 'musl.patch').

          - Checksums are used to verify the integrity of downloaded files. If
            the checksums do not match, moss will assume that the file is
            corrupted. To re-download the sources, use 'moss d [pkg]'.

          - The checksums can be generated by running 'moss c' inside the
            package directory.

      - strip: this key can optionally be set to 'false' to disable stripping
        binaries for a certain package. This is used to avoid breaking certain
        packages (e.g. musl). Most packages will not need this, and it can be
        omitted in such cases.

  - [deps] provides dependency information. Each dependency is given as a
    separate key, and the value given to each is a version requirement. These
    dependencies are treated as runtime dependencies.

      - In this example, 'libelf' depends on any version of 'zlib'.

      - Version requirements aren't really implemented yet (nor are they used by
        any official packages at the moment). For now, just use "*" as the
        version requirement (which means that any version is acceptable).

  - [mkdeps] is just like [deps], but dependencies provided here are assumed to
    only be necessary at compile time. In our example, 'libelf' requires a
    working install of 'pkgconf' in order to build properly.


=== Build $[022]

The build script is an executable file (typically a shell script) that is run
by the package manager to build a package. It is executed from within the
build directory and provided these arguments:

  - $1 is the absolute path to the destdir ($HOME/.cache/moss/build/[pkg]/dest).
  - $2 is the package version.


Package files must be installed to the destdir ($1):

  - For commands such as 'make install', set DESTDIR=$1.

  - When manually copying files, create the necessary directories with
    'mkdir -pv $1/[dir]' and prefix all paths with '$1'.


As an example, this is what the build script for 'libelf' looks like:

--------------------------------------------------------------------------------

  $ cat /var/repo/core/libelf/build

  #!/bin/sh -e

  patch -p1 < musl.patch

  # Build sometimes forces -Werror.
  export CFLAGS="$CFLAGS -Wno-error"

  sh ./configure \\
      --prefix=/usr \\
      --disable-symbol-versioning \\
      --disable-debuginfod \\
      --disable-libdebuginfod \\
      --disable-nls \\
      ac_cv_c99=yes # Override check for Clang.

  # Utility functions that need argp and fts, not strictly necessary
  # for the library to function
  :>libdwfl/argp-std.c
  :>libdwfl/linux-kernel-modules.c

  printf '%s\\n' "all:" "install:" > src/Makefile

  make
  make DESTDIR="$1" install

--------------------------------------------------------------------------------


=== Configuration $[030]
=======================

Moss is configured via a configuration file located at /etc/moss.toml. Here is
an explanation of the available configuration keys:


=== Repo search path $[031]

The key 'path' is an array that defines the directories that moss should look
for packages in. It looks like this by default:

--------------------------------------------------------------------------------

  path = [
      "/var/repo/core",
      "/var/repo/extra",
  ]

--------------------------------------------------------------------------------


To add a custom repo to the search path, simply add the full path to the array.
For example, if you have a package repository at ~/custom-repo, with a structure
something like this:

--------------------------------------------------------------------------------

  custom-repo
  |--- package1
  |    |--- build
  |    `--- package.toml
  |--- package2
  |    |--- build
  |    `--- package.toml
  `--- package3
       |--- build
       `--- package.toml

--------------------------------------------------------------------------------


You could add this to the repo search path (replace xxxxx with your username):

--------------------------------------------------------------------------------

  path = [
      "/var/repo/core",
      "/var/repo/extra",
      "/home/xxxxx/custom-repo"
  ]

--------------------------------------------------------------------------------


And you should now be able to do the following:

--------------------------------------------------------------------------------

  $ moss b package1 package2 package3

--------------------------------------------------------------------------------


=== Other config keys $[032]

Some information about the other available configuration keys:

  - verbose_builds: if this is set to true, build output for _all_ packages will
    be printed to stdout. If false, you can still enable build output by adding
    the 'v' flag to a build command (e.g. moss vb gcc). This key is required.

  - strip: if this is set to true, moss will strip unneeded symbols from binary
    files, and if set false, moss will not strip symbols from binaries. Specific
    packages can override this by setting meta.strip in package.toml. This key
    is required.

  - su_command: by default, moss will look for one of three commands to escalate
    privileges, in this order: 'sudo', 'doas', 'ssu'. If you want to use a
    different command for privilege escalation, you can provide that here; just
    note that you cannot use this to pass additional arguments to the privilege
    escalation utility. This key is optional.

  - cache_dir: moss normally performs builds and stores cache files in
    ~/.cache/moss but if you want a different location to be used (e.g.
    /tmp/moss) you can set that here. This key is optional.


=== APPENDIX $[100]
==================

This is some additional info that's just here for documentation purposes.


=== About $ARC_PATH $[110]

The environment variable $ARC_PATH was used in arc versions up to 0.0.3 to tell
arc where to find package repositories. It functioned similarly to $PATH, where
the contents are a colon separated list of paths to search for packages. For
versions of arc starting with 0.0.4, see [#030](#030).

Using the same example repo from [#030](#030), you could have added ~/custom-repo to
$ARC_PATH with the following:

--------------------------------------------------------------------------------

  $ export ARC_PATH=$HOME/custom-repo:$ARC_PATH

--------------------------------------------------------------------------------


And you would have been able to then do the following:

--------------------------------------------------------------------------------

  $ arc b package1 package2 package3

--------------------------------------------------------------------------------
