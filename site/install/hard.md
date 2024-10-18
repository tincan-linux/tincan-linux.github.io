+------------------------------------------------------------------------------+
|  Tin Can Linux installation guide (the hard way)                             |
+------------------------------------------------------------------------------+

This is a guide to building Tin Can Linux from the ground up by bootstrapping
the system yourself. This is a somewhat advanced process, so make sure you know
what you're doing! Unless you want to tinker with the process or need to build
for a different architecture, the easy method is recommended.


=== Contents
============

[[010](#010)] Bootstrapping the system
  [[011](#011)] Obtain sources
  [[012](#012)] Initial bootstrap
  [[013](#013)] Obtain the repositories
  [[014](#014)] Enter chroot environment
  [[015](#015)] Rebuild the system

[[020](#020)] Finish repository setup
  [[021](#021)] Obtain the repositories
  [[022](#022)] Set $ARC_PATH

[[030](#030)] Rebuild the system (again)
  [[031](#031)] Set compiler flags
  [[032](#032)] Rebuild all packages

[[040](#040)] Set up Linux kernel
  [[041](#041)] Download kernel sources
  [[042](#042)] Download firmware blobs
  [[043](#043)] Install required packages
  [[044](#044)] Configure the kernel
  [[045](#045)] Build and install the kernel

[[050](#050)] Install more packages
  [[051](#051)] Init scripts
  [[052](#052)] Bootloader
  [[053](#053)] Filesystem utilities
  [[054](#054)] Networking utilities
  [[055](#055)] Other optional tools

[[060](#060)] Make the system bootable
  [[061](#061)] Setup disks
  [[062](#062)] Install Tin Can to disk
  [[063](#063)] Install bootloader

[[070](#070)] Post installation
  [[071](#071)] Set root password
  [[072](#072)] Create a normal user
  [[073](#073)] Graphical environment
  [[074](#074)] Get counted


=== Bootstrapping the system $[010]
==================================

This step involves building a preliminary rootfs which you can enter via a
chroot environment.

=== Obtain sources $[011]

Clone the main git repo, [$/tincan-linux/tincan](https://github.com/tincan-linux/tincan), with the following command:

  $ git clone --recurse-submodules https://github.com/tincan-linux/tincan


This will obtain the bootstrap scripts along with the required submodules (the
bootstrap process uses [$/firasuke/mussel](https://github.com/firasuke/mussel) to build an initial cross compiler
toolchain)

Then cd into this directory.


=== Initial bootstrap $[012]

To perform the initial bootstrap, simply run:

  $ ./bootstrap


This (should) generate an initial root filesystem from which the distribution
will be built.


=== Obtain the repositories $[013]

For this initial rootfs, only [$/tincan-linux/repo-core](https://github.com/tincan-linux/repo-core) is required. Clone it
into the rootfs with:

  $ mkdir -pv sysroot/var/repo
  $ git clone https://github.com/tincan-linux/repo-core sysroot/var/repo/core


=== Enter chroot environment $[014]

The next step is to enter a chroot environment with sysroot/ as the new root.
For convenience, a modified version of kiss-chroot is provided; to enter the
chroot, execute the following (with root privileges):

  # ./arc-chroot sysroot


If all goes well, you should be presented with some lines of text that indicate
filesystems being mounted, followed by a shell prompt.


=== Rebuild the system $[015]

Once inside the chroot environment, you must rebuild all the packages, since
mussel does not use the latest versions, and also to start tracking packages
with arc, the package manager (see [@/wiki/arc](/wiki/arc) for more information on it)

The packages must be built in a specific order for everything to work as
expected. Perform the following (in this exact order):

  # arc sh
  # arc b linux-headers
  # arc b musl
  # arc b m4
  # arc b binutils
  # arc b gcc
  # arc b make
  # arc b busybox
  # arc b rustup
  # arc b arc


Hopefully, everything will go well. If not, exit the chroot, remove the sysroot/
directory, and start back at $[012].
