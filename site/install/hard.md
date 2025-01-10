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
  [[021](#021)] Set repo search path

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
  [[062](#062)] Create /etc/fstab
  [[063](#063)] Install Tin Can to disk
  [[064](#064)] Install bootloader

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

--------------------------------------------------------------------------------

  $ git clone --recurse-submodules https://github.com/tincan-linux/tincan

--------------------------------------------------------------------------------


This will obtain the bootstrap scripts along with the required submodules (the
bootstrap process uses [$/firasuke/mussel](https://github.com/firasuke/mussel) to build an initial cross compiler
toolchain)

Then cd into this directory.


=== Initial bootstrap $[012]

To perform the initial bootstrap, simply run:

--------------------------------------------------------------------------------

  $ ./bootstrap

--------------------------------------------------------------------------------


This (should) generate an initial root filesystem from which the distribution
will be built.


=== Obtain the repositories $[013]

There are two main official repos: [$/tincan-linux/repo-core](https://github.com/tincan-linux/repo-core) and
[$/tincan-linux/repo-extra](https://github.com/tincan-linux/repo-extra). Clone them into the rootfs with:

--------------------------------------------------------------------------------

  $ mkdir -pv sysroot/var/repo
  $ git clone https://github.com/tincan-linux/repo-core sysroot/var/repo/core
  $ git clone https://github.com/tincan-linux/repo-extra sysroot/var/repo/extra

--------------------------------------------------------------------------------


These repositories can be installed anywhere you want. If you decide to install
to a different location than /var/repo/ then you will need to make some minor
edits in step [[021](021)].


=== Enter chroot environment $[014]

The next step is to enter a chroot environment with sysroot/ as the new root.
For convenience, a modified version of kiss-chroot is provided; to enter the
chroot, execute the following (with root privileges):

--------------------------------------------------------------------------------

  # ./arc-chroot sysroot

--------------------------------------------------------------------------------


If all goes well, you should be presented with some lines of text that indicate
filesystems being mounted, followed by a shell prompt.


=== Rebuild the system $[015]

Once inside the chroot environment, you must rebuild all the packages, since
mussel does not use the latest versions, and also to start tracking packages
with arc, the package manager (see [@/wiki/arc](/wiki/arc) for more information on it).

First, you will need to make sure that the compiler is able to find all needed
header files for compiling packages. Right now, it is likely using an incorrect
include path that is based on the direcory where ./bootstrap was run. First,
find out what this path is by executing the following:

--------------------------------------------------------------------------------

  # gcc -v -x c -E /dev/null
 
--------------------------------------------------------------------------------


This will produce a bunch of diagnostic output about GCC. In this output, look
for the lines that are similar to the following (they should be somewhere in the
middle):

--------------------------------------------------------------------------------

  gcc version 13.2.0 (GCC)
  COLLECT_GCC_OPTIONS='-v' '-E' '-mtune=generic' '-march=x86-64'
   /lib/gcc/x86_64-linux-musl/13.2.0/cc1 -E -quiet -v /dev/null -mtune=generic
   -march=x86-64 -dumpbase null
  ignoring nonexistent directory "/home/xxxxx/tincan/sysroot/usr/local/include"
  ignoring nonexistent directory "/lib/gcc/x86_64-linux-musl/13.2.0/../../../../
  x86_64-linux-musl/include"
  ignoring nonexistent directory "/home/xxxxx/tincan/sysroot/usr/include"
  #include "..." search starts here:
  #include <...> search starts here:
   /lib/gcc/x86_64-linux-musl/13.2.0/include
  End of search list.

--------------------------------------------------------------------------------


The last nonexistent directory, /home/xxxxx/tincan/sysroot/usr/include, is what
we need. Create this directory and point it to /include (replace this with the
approrpiate path, based on the previous output):

--------------------------------------------------------------------------------

  # mkdir -pv /home/xxxxx/tincan/sysroot/usr/
  # ln -sv /include /home/xxxxx/tincan/sysroot/usr/include

--------------------------------------------------------------------------------


Now, if you re-run the gcc command from above, there should be an additional
directory under '#include<...>':

--------------------------------------------------------------------------------

  #include <...> search starts here:
   /home/xxxxx/tincan/sysroot/usr/include
   /lib/gcc/x86_64-linux-musl/13.2.0/include
  End of search list.

--------------------------------------------------------------------------------


Now, we are ready to build the packages.

The packages must be built in a specific order for everything to work as
expected. Perform the following (in this exact order):

--------------------------------------------------------------------------------

  # arc b linux-headers
  # arc b musl
  # arc b m4
  # arc b binutils
  # arc b gcc
  # arc b make
  # arc b busybox
  # arc b rustup
  # rustup default stable
  # arc b arc

  Or in a single command, so you can walk away and come back later:

  # arc yb linux-headers \&\& arc yb musl \&\& arc yb m4 \&\& arc yb binutils \\
    \&\& arc yb gcc \&\& arc yb make \&\& arc yb busybox \&\& arc yb rustup \\
    \&\& rustup default stable \&\& arc yb arc

  Remove the compatibility symlink to /include:

  # rm -rf /home/xxxxx

--------------------------------------------------------------------------------


Hopefully, everything will go well. If not, exit the chroot, remove the sysroot/
directory, and start back at $[012].

From this point on, the guide is very similar to the easy method. As the
maintainer of Tin Can Linux, this is where I would clean up some stuff and
create a tarball of the rootfs.


=== Finish repository setup $[020]
=================================

In this step, we will finish proper setup of the official repositories.


=== Set repo search path $[021]

Arc has a configuration file at /etc/arc.toml that allows you to set several
options, including the repo search path. By default this includes the
directories '/var/repo/core' and '/var/repo/extra'.

First, make sure that /etc/arc.toml exists. If not, create it:

--------------------------------------------------------------------------------

# List of repositories that arc should search for packages. The full path to
# each repository should be provided.
path = [
    "/var/repo/core",
    "/var/repo/extra",
]

# Controls whether to display output from build scripts to the terminal. If true
# build script output will be shown for all bulds, and if false it will only be
# shown if the 'v' flag is provided at the command line.
verbose_builds = false

# Controls whether to strip binaries. If true, all packages will have their
# binaries stripped by default, and if false binaries will not be stripped by
# default. Individual packages can override this setting by setting
# the 'strip' key under the [meta] section in package.toml.
strip = true

# Specify a command to use for privelege escalation. If this is not set, arc
# will look for and use one of 'sudo', 'doas', or 'ssu' (in that order). Also
# assumes the syntax is similar to that of 'sudo'.
# su_command = "doas"

# Specify a different cache directory for builds. If this is not set, arc will
# use ~/.cache/arc by default.
# cache_dir = "/tmp/arc"

--------------------------------------------------------------------------------


If you decided to install the repos to a different location, edit these lines in
/etc/arc.toml:

--------------------------------------------------------------------------------

  path = [
      "/var/repo/core",
      "/var/repo/extra",
  ]

--------------------------------------------------------------------------------


See [@/wiki/arc](/wiki/arc) to learn more about configuring arc.


=== Rebuild the system (again) $[030]
====================================

In order to ensure that all packages have been built with the latest versions
of dependencies, and to install some packages that were not included the first
time, we will rebuild all the packages in the system.


=== Set compiler flags $[031]

It may be helpful to set the following compiler flags:

  - CFLAGS: Setting one of '-O2', '-Os', or '-O3' can be used to optimize the
    binaries that are produced.

  - CFLAGS: Setting '-pipe' can speed up compilation at the cost of higher
    memory usage.

  - CFLAGS: Setting '-march=native' makes the compiler use processor-specific
    optimizations. Omit this if you are planning to create a portable install
    and also note that this flag can occasionally break some packages.

  - MAKEFLAGS: Setting '-j$(nproc)' will enable builds to use all available CPU
    cores. However, this can make it harder to track down compile errors.


With this in mind, add the following lines to /etc/profile:

--------------------------------------------------------------------------------

  export CFLAGS="-O3 -pipe -march=native"
  export CPPFLAGS="$CFLAGS"
  export CXXFLAGS="$CFLAGS"
  export MAKEFLAGS="-j$(nproc)"

--------------------------------------------------------------------------------


And source it to apply the changes to your current shell:

--------------------------------------------------------------------------------

  # source /etc/profile

--------------------------------------------------------------------------------


=== Rebuild all packages $[032]

Run the following to rebuild all packages (and install some new ones):

--------------------------------------------------------------------------------

  # arc b arc binutils bison busybox certs flex gcc git gmp \\
    linux-headers m4 make mpc mpfr musl rustup zlib

--------------------------------------------------------------------------------


=== Set up Linux kernel $[040]
=============================

This step will detail how to configure, build, and install the Linux kernel.


=== Download kernel sources $[041]

First, you will need to select a kernel. There are a few common variants:

  - Vanilla kernel. Nothing special.
  - Hardened kernel. A fork of the kernel focused on security.
  - Long term kernel. Designed for long-term support and server usage.
  - Zen kernel. An effort by kernel hackers to improve the vanilla kernel.


There are also some other less common kernels; see [this Arch Wiki article](https://wiki.archlinux.org/title/Kernel) for
more information.

The kernel sources can then be obtained from one of these URLs, depending on
which variant you want to use:

  - Vanilla: https://www.kernel.org/
  - Hardened: https://github.com/anthraxx/linux-hardened
  - Long term: https://www.kernel.org/
  - Zen: https://github.com/zen-kernel/zen-kernel


Then download and extract it with the following:

--------------------------------------------------------------------------------

  # curl -fLO [KERNEL_SOURCE]
  # tar xf [KERNEL_SOURCE]
  # cd [KERNEL_SOURCE]

--------------------------------------------------------------------------------


=== Download firmware blobs $[042]

It is likely that your hardware may need some additional firmware that isn't
provided in the kernel. Some of this firmware can be found at this mirror:

  https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git


Other firmware, such as Intel Microcode, may need to be obtained from other
sources. See [@/wiki/kernel](/wiki/kernel) for more detailed info on selecting and
obtaining the correct firmware for your hardware.

After obtaining the necessary firmware, extract the tarballs and copy required
files to /usr/lib/firmware:

--------------------------------------------------------------------------------

  # tar xf [FIRMWARE]
  # mkdir -pv /usr/lib/firmware
  # cp -R ./path/to/blob /usr/lib/firmware/

--------------------------------------------------------------------------------


=== Install required packages $[043]

Some additional packages are required beyond the ones installed so far in order
to compile the kernel. Install them now:

--------------------------------------------------------------------------------

  # arc b libelf pkgconf

  For menuconfig support (highly recommended):

  # arc b ncurses

--------------------------------------------------------------------------------


=== Configure the kernel $[044]

This is one of the most critical (and also tricky) steps of the process. Doing
something wrong here can prevent your system from booting. While there are many
ways to go about configuring the kernel, I will be presenting one method here
which has worked for me.

Start with a default config, which should include some sane default settings,
then open the configuration menu:

--------------------------------------------------------------------------------

  # make defconfig
  # make menuconfig

--------------------------------------------------------------------------------


This will open a blue TUI where you can configure the kernel options. While
there isn't a one-size-fits-all procedure for this part of the process, here
are some bits that I found are important to note:

  - YOU NEED GRAPHICS DRIVERS!!! One of the most common pitfalls with kernel
    configuration is leaving out a graphics driver, usually for the framebuffer.
    Your system probably will not boot without these options:

      - CONFIG_FB
      - CONFIG_FB_CORE
      - CONFIG_FB_DEVICE
      - CONFIG_FB_NOTIFY
      - CONFIG_FRAMEBUFFER_CONSOLE
      - CONFIG_SYSFB
      - CONFIG_SYSFB_SIMPLEFB

    Additionally, some combination of these may be needed:

      - CONFIG_FB_EFI
      - CONFIG_FB_VESA
      - CONFIG_FB_VGA16
      - CONFIG_DRM_FBDEV_EMULATION

    There may be some other ones beyond these that I haven't covered. A good
    check is to do 'grep "FB" .config \&\& grep "FRAMEBUFFER" .config' to see what
    other framebuffer-related graphics drivers are available.

    In additon to these, graphics drivers for specific graphics cards are also
    probably needed, such as:

      - CONFIG_DRM_I915 for Intel graphics cards
      - CONFIG_DRM_AMDGPU for AMD graphics cards
      - CONFIG_DRM_SIMPLEDRM + additional firmware for NVIDIA graphics cards;
        see https://wiki.archlinux.org/title/NVIDIA

  - Specific hardware, such as WiFi cards, touchpads, etc. may also require
    additional kernel options to be enabled. A good way to see what's needed is
    to run 'lspci' and 'lsmod' from the live environment:

      - 'lspci' gives information on installed hardware devices, so that you can
        look up what drivers are needed for your specific model.

      - 'lsmod' tells you what kernel modules are currently loaded. This can
        point out some options that need to be enabled in the kernel (just
        google the module name to find its equivalent in the kernel menuconfig)

  - The Gentoo Wiki is a great source of information about kernel configuration.
    Try looking up your model or laptop brand to see if it has a dedicated wiki
    page with kernel configuration info. For example, I found [this one](https://wiki.gentoo.org/wiki/Framework_Laptop_13) for the
    Framework Laptop that saved me a lot of trouble.

  - If you're stuck, Reddit is your friend! There's probably someone out there
    who had the same problem you do, and if not, there are plenty of communities
    where you could ask a question and see if someone can help you get to the
    answer.

  - You can also ask a question on [the "forum" (github discussions)](https://github.com/orgs/tincan-linux/discussions).

  - Also check out [@/wiki/kernel](/wiki/kernel) which serves as a place for all things
    related to kernel configuration. Tips, tricks, etc. Although it's looking
    kinda empty right now :(


Once that's done, change all modules to baked-in (since modules are yucky):

--------------------------------------------------------------------------------

  # sed -i 's/=m/=y/g' .config

--------------------------------------------------------------------------------


=== Build and install the kernel $[045]

Execute the following commands to build and install the kernel:

--------------------------------------------------------------------------------

  # make
  # make install
  # mv /boot/vmlinuz /boot/vmlinuz-[VERSION]
  # mv /boot/System.map /boot/System.map-[VERSION]

--------------------------------------------------------------------------------


The 'make install' step may throw a LILO error. This is normal and doesn't
indicate any problems in the build or install process.


=== Install more packages $[050]
===============================

In this section, we will install some more pacakges -- the final ones needed
before we can make the system bootable.


=== Init scripts $[051]

The package 'tincan-base' provides a tiny init script, [$/tincan-linux/init](https://github.com/tincan-linux/init) along
with some files unique to Tin Can Linux. Install it with the following:

--------------------------------------------------------------------------------

  # arc b tincan-base

--------------------------------------------------------------------------------


=== Bootloader $[052]

Tin Can Linux uses [$/limine-bootloader/limine](https://limine-bootloader.org) instead of GRUB ([why?](/wiki/bootloader )) -- install
it with this command:

--------------------------------------------------------------------------------

  # arc b limine

--------------------------------------------------------------------------------


=== Filesystem utilities $[053]

While not strictly required, filesystem utilities may be needed for checking
disks with 'fsck':

--------------------------------------------------------------------------------

  # arc b e2fsprogs dosfstools

--------------------------------------------------------------------------------


=== Networking utilities $[054]

Again, these are not strictly required, but I assume most people want WiFi:

--------------------------------------------------------------------------------

  # arc b dhcpcd openresolv libudev-zero eiwd

--------------------------------------------------------------------------------


=== Other optional tools $[055]

These are some other utilities that you may find useful:

--------------------------------------------------------------------------------

  # arc b ssu

--------------------------------------------------------------------------------


=== Make the system bootable $[060]
==================================

Finally, it's time to boot your new, shiny Linux system! Go ahead and exit the
chroot:

--------------------------------------------------------------------------------

  # exit

--------------------------------------------------------------------------------


=== Setup disks $[061]

The first step is to partition the disk that you will be installing Tin Can
Linux to. You will need at least two partitions (one for boot, one for root) but
this guide will use three (boot, root, and swap).

First, identify your disk (I use /dev/sda in these examples). Then, for a UEFI
install, run these commands as root to partition the disks:

--------------------------------------------------------------------------------

  # parted /dev/sda -- mklabel gpt
  # parted /dev/sda -- mkpart root ext4 512MB -8GB
  # parted /dev/sda -- mkpart swap linux-swap -8GB 100%
  # parted /dev/sda -- mkpart ESP fat32 1MB 512MB
  # parted /dev/sda -- set 3 esp on

--------------------------------------------------------------------------------


And for a BIOS install:

--------------------------------------------------------------------------------

  # parted /dev/sda -- mklabel msdos
  # parted /dev/sda -- mkpart primary 512MB -8GB
  # parted /dev/sda -- mkpart primary linux-swap -8GB 100%
  # parted /dev/sda -- mkpart primary fat32 1MB 512MB
  # parted /dev/sda -- set 3 boot on

--------------------------------------------------------------------------------


Feel free to change these partition sizes as you see fit, or add other
partitions (for example, some users like to have a separate home partition).

Next, format these partitions appropriately:

--------------------------------------------------------------------------------

  # mkfs.ext4 -L tincan /dev/sda1
  # mkswap -L swap /dev/sda2
  # mkfs.fat -F 32 -n boot /dev/sda3

--------------------------------------------------------------------------------


Finally, mount the root and boot partitions:

--------------------------------------------------------------------------------

  # mount /dev/sda1 /mnt
  # mkdir -pv /mnt/boot
  # mount /dev/sda3 /mnt/boot

--------------------------------------------------------------------------------


=== Create /etc/fstab $[062]

Now that the disks are set up, you will need to create the /etc/fstab file. As
the root user, create sysroot/etc/fstab (we're not in the chroot anymore) with
these contents, assuming the partitioning scheme above:

--------------------------------------------------------------------------------

# Begin /etc/fstab

# file system  mount-point     type      options             dump  fsck

LABEL=boot     /boot           vfat      defaults             0     0
LABEL=tincan   /               ext4      defaults             0     1
LABEL=swap     swap            swap      pri=1                0     0
proc           /proc           proc      nosuid,noexec,nodev  0     0
sysfs          /sys            sysfs     nosuid,noexec,nodev  0     0
devpts         /dev/pts        devpts    gid=5,mode=620       0     0
tmpfs          /run            tmpfs     defaults             0     0
devtmpfs       /dev            devtmpfs  mode=0755,nosuid     0     0
tmpfs          /dev/shm        tmpfs     nosuid,nodev         0     0
cgroup2        /sys/fs/cgroup  cgroup2   nosuid,noexec,nodev  0     0

# End /etc/fstab

--------------------------------------------------------------------------------


=== Install Tin Can to disk $[063]

To install your shiny new distro, simply copy the contents of sysroot/ to /mnt:

--------------------------------------------------------------------------------

  # cp -R sysroot/* /mnt/

--------------------------------------------------------------------------------


=== Install bootloader $[064]

The final step is to install the bootloader. First, re-enter the chroot with:

--------------------------------------------------------------------------------

  # ./arc-chroot /mnt

--------------------------------------------------------------------------------


Then, for a BIOS install (run from the chroot):

--------------------------------------------------------------------------------

  # cp /usr/share/limine/limine-bios.sys /boot/
  # limine bios-install /dev/sda

--------------------------------------------------------------------------------


Or for a UEFI install:

--------------------------------------------------------------------------------

  # mkdir -pv /boot/EFI/BOOT
  # cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/

--------------------------------------------------------------------------------


Next, create /boot/limine.conf with this content:

--------------------------------------------------------------------------------

  timeout: 5

  /Tin Can Linux
      protocol: linux
      kernel_path: boot():/vmlinuz-[VERSION]
      kernel_cmdline: root=UUID=xxxx-xx--xxx ro loglevel=3 rootwait quiet

--------------------------------------------------------------------------------


To get the UUID of your root partition, simply run 'blkid'. You can also replace
the 'UUID=xxxx' part with '/dev/sda1' but this may be less reliable than using
the disk's UUID. If there's a kernel panic when booting that somewhat resembles
'VFS not syncing' then try replacing the UUID with /dev/sda1.


And that's it! Tin Can Linux should now be installed. Exit the chroot, then
reboot to enter your shiny new system:

--------------------------------------------------------------------------------

  # exit
  # reboot now

--------------------------------------------------------------------------------


Oh, what's that? It didn't work? Here are some reasons why:

  - It doesn't even reach the bootloader:
      - You may have secure boot enabled.

  - It gets to the bootloader, but hangs at a black screen:
      - Congratulations, your kernel configuration didn't work! Go back to $[044]
        and play around with some stuff.


If it did work, and you reach a login prompt, then congratulations! Enjoy your
very own scrappy distro!


=== Post installation $[070]
===========================

Now that you've booted into Tin Can Linux, here's some stuff you can do:


=== Set a root password $[071]

PLEASE DO THIS. IT'S A SECURITY RISK IF YOU DON'T.

--------------------------------------------------------------------------------

  # passwd root

--------------------------------------------------------------------------------


=== Create a normal user $[072]

Avoid accidentally wrecking your system (not that it will stop dedicated users):

--------------------------------------------------------------------------------

  # adduser [USER]
  # passwd [USER]

--------------------------------------------------------------------------------


=== Graphical environment $[073]

Install the wayland libraries (this will get everything):

--------------------------------------------------------------------------------

  # arc b wlroots

--------------------------------------------------------------------------------


Compositors, terminals, etc are not provided. You will have to obtain and build
these yourself. Here are some of my recommendations, though the choice is
entirely up to you:

  - Compositor: river, dwl, owl
  - Terminal: foot, havoc
  - Status bar: yambar


And to install a minimal web browser:

--------------------------------------------------------------------------------

  # arc b netsurf

--------------------------------------------------------------------------------


Additional information about installing a graphical environment can be found at
[@/wiki/graphical](/wiki/graphical).


=== Get counted $[074]

Now that you're a user of Tin Can Linux, head on over to [@/register](/register) to get
counted and immortalize yourself in the Tin Can Linux user list! You can also
see who else is using Tin Can Linux.


=== Resources
=============

These are some resources used throughout the making of this guide, in no
particular order:

  - https://www.linuxfromscratch.org
  - https://github.com/firasuke/mussel
  - https://kisslinux.github.io
  - https://nixos.org/manual/nixos/stable
  - https://wiki.archlinux.org/title/Limine
  - https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel
  - https://wiki.gentoo.org/wiki/Framework_Laptop_13
  - https://wiki.archlinux.org/title/Kernel
  - https://wiki.archlinux.org/title/NVIDIA
