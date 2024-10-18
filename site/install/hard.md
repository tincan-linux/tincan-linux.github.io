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

From this point on, the guide is very similar to the easy method. As the
maintainer of Tin Can Linux, this is where I would clean up some stuff and
create a tarball of the rootfs.


=== Finish repository setup $[020]
=================================

In this step, we will finish proper setup of the official repositories.


=== Obtain the repositories $[021]

Temporarily exit the chroot environment, as we haven't yet installed git inside
the rootfs:

  # exit


This will unmount the filesystems that were previously mounted and drop you back
into the host machine's shell. From there, we can clone the remaining repos:

  $ git clone https://github.com/tincan-linux/repo-extra sysroot/var/repo/extra
  $ (repo-xorg is not here yet)


Now re-enter the chroot:

  # ./arc-chroot


=== Set $ARC_PATH $[022]

The environment variable $ARC_PATH is used by arc to determine where to find the
repositories. This allows you to install repositories to any location on the
system.

To let arc know where to find the official repositories, add this line to
/etc/profile:

  export ARC_PATH=/var/repo/core:/var/repo/extra


From now on, arc will look for packages in /var/repo/core and /var/repo/extra.
See [@/wiki/arc](/wiki/arc) to learn more about how $ARC_PATH works.


=== Rebuild the system (again) $[030]
====================================

In order to ensure that all packages have been built with the latest versions
of dependencies, and to install some packages that were not included the first
time, we will rebuild all the packages in the system.


=== Set compiler flags $[031]

It may be helpful to set the following compiler flags:

  - CFLAGS: Setting one of '-O2', '-Os', or '-O3' can be used to optimize the
    binareies that are produced.

  - CFLAGS: Setting '-pipe' can speed up compilation at the cost of higher
    memory usage.

  - CFLAGS: Setting '-march=native' makes the compiler use processor-specific
    optimizations. Omit this if you are planning to create a portable install
    (e.g. on a thumb drive)

  - MAKEFLAGS: Setting '-j$(nproc)' will enable builds to use all available CPU
    cores. However, this can make it harder to track down compile errors.


With this in mind, add the following lines to /etc/profile:

  export CFLAGS="-O3 -pipe -march=native"
  export CPPFLAGS="$CFLAGS"
  export CXXFLAGS="$CFLAGS"
  export MAKEFLAGS="-j$(nproc)"


=== Rebuild all packages $[032]

Run the following to rebuild all packages (and install some new ones):

  # arc b arc binutils bison busybox certs flex gcc git gmp \\
    linux-headers m4 make mpc mpfr musl rustup zlib


=== Set up Linux Kernel $[040]
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

  # curl -fLO [KERNEL_SOURCE]
  # tar xf [KERNEL_SOURCE]
  # cd [KERNEL_SOURCE]


=== Download firmware blobs $[042]

It is likely that your hardware may need some additional firmware that isn't
provided in the kernel. Some of this firmware can be found at this mirror:

  https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git


Other firmware, such as Intel Microcode, may need to be obtained from other
sources. See [@/install/kconfig](/install/kconfig) for more detailed info on selecting and
obtaining the correct firmware for your hardware.

After obtaining the necessary firmware, extract the tarballs and copy required
files to /usr/lib/firmware:

  # tar xf [FIRMWARE]
  # mkdir -pv /usr/lib/firmware
  # cp -R ./path/to/blob /usr/lib/firmware/


=== Install required packages $[043]

Some additional packages are required beyond the ones installed so far in order
to compile the kernel. Install them now:

  # arc b libelf pkgconf

  For menuconfig support (highly recommended):

  # arc b ncurses


=== Configure the kernel $[044]

This is one of the most critical (and also tricky) steps of the process. Doing
something wrong here can prevent your system from booting. While there are many
ways to go about configuring the kernel, I will be presenting one method here
which has worked for me.

Start with a default config, which should include some sane default settings:

  # make defconfig


Next, start the menuconfig:

  # make menuconfig


This will open a blue TUI where you can configure the kernel options. While
there isn't a one-size-fits-all procedure for this part of the process, here
are some bits that I found are important to note:

  - YOU NEED GRAPHICS DRIVERS. One of the most common pitfalls with kernel
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
      - CONFIG_DRM_SIMPLEDRM + additional firmware for NVIDIA graphics cards
        (see https://wiki.archlinux.org/title/NVIDIA)

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

  - Also check out [@/install/kconfig](/install/kconfig) which serves as a place for all things
    related to kernel configuration. Tips, tricks, etc. Although it's looking
    kinda empty right now :(


Once that's done, change all modules to baked-in (since modules are yucky):

  # sed -i 's/=m/=y/g' .config


=== Build and install the kernel $[045]

Execute the following commands to build and install the kernel:

  # make
  # make install
  # mv /boot/vmlinuz /boot/vmlinuz-[VERSION]
  # mv /boot/System.map /boot/System.map-[VERSION]


The 'make install' step may throw a LILO error. This is normal and doesn't
indicate any problems in the build or install process.
