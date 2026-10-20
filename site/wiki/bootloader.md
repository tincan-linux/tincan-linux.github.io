+------------------------------------------------------------------------------+
|  About the bootloader.                                                       |
+------------------------------------------------------------------------------+

Tin Can Linux uses [$/limine-bootloader/limine](https://limine-bootloader.org) instead of GRUB or other
common bootloaders to maintain simplicity (and a few other reasons).

=== Contents
============

[[010](#010)] Installation

[[020](#020)] Configuration

[[030](#030)] Rationale


=== Installation $[010]
======================

Install the bootloader with moss.

--------------------------------------------------------------------------------

  $ moss b limine

--------------------------------------------------------------------------------


For a BIOS install (run as root):

--------------------------------------------------------------------------------

  # cp /usr/share/limine/limine-bios.sys /boot/
  # limine bios-install /dev/sda

--------------------------------------------------------------------------------


Or for a UEFI install (run as root):

--------------------------------------------------------------------------------

  # mkdir -pv /boot/EFI/BOOT
  # cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/

--------------------------------------------------------------------------------


=== Configuration $[020]
=======================

The configuration file is located at /boot/limine.conf and should look something
like this if you followed the installtion guide:

--------------------------------------------------------------------------------

  timeout: 5

  /Tin Can Linux
      protocol: linux
      kernel_path: boot():/vmlinuz-[VERSION]
      kernel_cmdline: root=UUID=xxxx-xx--xxx ro loglevel=3 rootwait quiet

--------------------------------------------------------------------------------


This should typically be enough for a working installation of Tin Can Linux,
though there are some things to note about this basic configuration:

  - kernel_cmdline: while it's better to use the disk UUID if possible, this may
    not always work. Alternatively, the disk can be specified as
    `root=/dev/sdXY` if you know that the automatically assigned disk label
    won't change. This is typically the case if there is only one disk present.


Additionally, there are some potentially useful global config keys:

  - quiet: this can be added to hide all output (including the menu) except
    important warnings and panics. Purely aesthetic, and pressing any key brings
    back the menu and pauses the timeout as usual.

  - verbose: show more output, useful for debugging problems.

  - default_entry: the name or index of the entry to select by default.

  - wallpaper: can use a background image for fun. Supports BMP, PNG, JPEG, QOI.


And some useful local configuration keys (per-entry):

  - comment: displayed when an entry is highlighted.

  - module_path: if you're using an initramfs (which Tin Can does not use by
    default, since kernel modules are baked in), you can specify that here.


A customized config file might therefore look something like this:

--------------------------------------------------------------------------------

  timeout: 5
  quiet: yes
  wallpaper: boot():/wallpaper.png

  /Tin Can Linux
      protocol: linux
      kernel_path: boot():/vmlinuz-7.2.11-zen1
      kernel_cmdline: root=/dev/sda1 ro loglevel=3 rootwait quiet
      module_path: boot():/initrd
      comment: A really awesome tiny Linux distribution

--------------------------------------------------------------------------------


This is not at all an exhaustive list of the configuration options, just the
ones I think seemed relevant. The full documentation for the config file can be
found at [https://github.com/Limine-Bootloader/Limine/blob/v12.x/CONFIG.md].


=== Rationale $[030]
===================

I originally picked limine because it's simple, reasonably suckless, and not
exactly your usual bootloader. In other words, it fits perfectly with the
general themes of Tin Can as a distro.

Also, GRUB wasn't working back when I was first developing Tin Can. Limine did.