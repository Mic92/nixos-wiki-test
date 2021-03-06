---
title: Installing NixOS from a USB stick
permalink: /Installing_NixOS_from_a_USB_stick/
---

Since NixOS 14.11 the installer ISO is hybrid. This means it is bootable on both CD and USB drives. It also boots on EFI systems, like most modern motherboards and Mac\* systems. Please read the [\#Hybrid image](/#Hybrid_image "wikilink") section on how to burn the ISO.

For NixOS 14.04 and earlier, please read [\#Using Unetbootin](/#Using_Unetbootin "wikilink") and the other sections below.

Hybrid image
------------

NOTE: This section applies to NixOS 14.11. For NixOS 14.04, please refer to [\#Using Unetbootin](/#Using_Unetbootin "wikilink").

First, download a [NixOS ISO image](http://nixos.org/nixos/download.html) or [create a custom ISO](/Creating_a_Nix_OS_live_CD "wikilink"). Then plug in a USB stick large enough to accomodate the image. Then follow the platform instructions:

-   On Linux:
    1.  Find the right device with `fdisk -l`, let's say `/dev/sdb`.
    2.  Burn with: `cp nixos-xxx.iso /dev/sdb`

    Note: do not use /dev/sdb1 or partitions of the disk, use the whole disk /dev/sdb.

-   On OS X:
    1.  Find the right device with `diskutil list`, let's say `/dev/disk2`.
    2.  Unmount (but don't eject) if needed with `diskutil unmountDisk /dev/disk2`.
    3.  Burn with: `sudo cp nixos-xxx.iso /dev/disk2` (please be very careful, overwriting your root disk is a bad idea).
    4.  Write to the full disk, not a partition.

    You could use Disk Utility to write the image but then you'd need to convert it first to a compatible format somehow.

-   On Windows:
    1.  Download [USBwriter](http://sourceforge.net/projects/usbwriter/).
    2.  Start USBwriter.
    3.  Choose the downloaded ISO as 'Source'
    4.  Choose the USB drive as 'Target'
    5.  Click 'Write'
    6.  When USBwriter has finished writing, safely unplug the USB drive.

This should suffice to boot the installer. Otherwise look for other alternatives below.

UEFI note
---------

The below is for BIOS installation. For UEFI installation see [the manual](http://nixos.org/nixos/manual/#sec-uefi-installation).

Using Unetbootin
----------------

It is possible to install NixOS from a USB stick, rather than from a CD. This is useful if you want to install NixOS on a machine that doesn't have a CD-ROM drive (such as most netbooks), or if you don't want to waste a blank CD. Here is how to do it:

1.  Download a [NixOS ISO image](http://nixos.org/nixos/download.html) or [create a custom ISO](/Creating_a_Nix_OS_live_CD "wikilink").
2.  Obtain a USB stick formatted with the VFAT/FAT32 filesystem with enough free disk space to hold the contents of the ISO image. Note that it's not necessary to erase the USB stick.
3.  Install [UNetbootin](http://unetbootin.sourceforge.net/), a tool that allows you to create a bootable USB stick from an ISO image. UNetbootin runs on both Linux and Windows. If you already have Nix/NixOS, you can install it by running `nix-env -i unetbootin`. [Other tools](http://en.wikipedia.org/wiki/List_of_tools_to_create_Live_USB_systems) may also work.
4.  Insert the USB stick, start UNetbootin, select the ISO file and target USB drive, and press Ok. This copies the contents of the ISO to the USB stick and installs the GRUB boot loader.
5.  You should now be able to boot NixOS from the USB stick, and perform the installation as usual.

NOTE: For EFI support you may need to change the label of the FAT filesystem. You can do this on Linux with mlabel, and on OS X with diskutil: \`sudo diskutil rename OLDNAME NIXOS_ISO\`.

Using syslinux
--------------

### Using syslinux.cfg

If you have trouble booting from a USB drive or sdcard with Unetbootin (I just got a blinking cursor instead of a bootloader on an Asus eee 1000) the procedure described on <http://knoppix.net/wiki/Bootable_USB_Key> should work. After preparing the sdcard with mkdiskimage and syslinux you can mount it and copy all the content from the NixOS iso:

    $ mount -o loop ~/Downloads/nixos-graphical-0.1pre27337-i686-linux.iso /media/iso
    $ rsync -av --progress /media/iso/ /media/sd-card/

The syslinux.cfg needs to be created manually, but the details can easily be derived from the grub.cfg on the NixOS install cd.

I specified the root device by UUID, to get the UUID:

    blkid /dev/mmcblk0p1

The relevant section from the live cd: /media/sd-card/boot/grub/grub.cfg

    menuentry "NixOS Installer / Rescue" {
      linux /boot/bzImage init=/nix/store/r7xhnzymi1ll49r4glf1dwr5y1alx0bl-system/init root=LABEL=NIXOS_INSTALL_CD_0.1pre27337 splash=verbose vga=0x317
      initrd /boot/initrd
    }

can be used in the syslinux config file: /media/sd-card/syslinux.cfg (you should just need to update the init path and the root UUID / LABEL or device path)

    DEFAULT linux
    LABEL linux
      SAY Now booting the kernel from SYSLINUX...
      KERNEL /boot/bzImage
      APPEND init=/nix/store/r7xhnzymi1ll49r4glf1dwr5y1alx0bl-system/init root=UUID=509C-63E2 ro initrd=/boot/initrd splash=verbose

Alternate method
----------------

1.  Download the ISO image from <http://nixos.org/nixos/download.html>.
2.  Prepare your USB stick. If it isn't yet partitioned to your liking, create a bootable partition on `/dev/sdb1`:
        fdisk /dev/sdb

    And format it with:

        mkdosfs /dev/sdb1

    Add ext2 partitions if you like. If you want both 32-bit and 64-bit NixOS, you need at least two partitions. You can directly use the entire disk without partition if that's what you like. Once you have your VFAT (dosfs) block device, you should give it a label, within a 11-character limit, e.g.:

        dosfslabel /dev/sdb1 NIXBOOT

3.  Mount the bootable USB partition with:
        mkdir -p /media/NIXBOOT
        mount /dev/sdb1 /media/NIXBOOT

4.  Mount the ISO image with:
        mkdir -p /media/iso
        mount -o loop ~/Downloads/nixos-graphical-0.2pre4463_5e88e9b-c877f45-x86_64-linux.iso /media/iso

5.  Copy the contents of the NixOS ISO (`/media/iso`) to your USB stick (`/media/NIXBOOT`) with:
        rsync -av --progress /media/iso/ /media/NIXBOOT/

6.  Install a complete and recent GRUB to the USB stick with:
        grub-install /dev/sdb --root-directory=/media/NIXBOOT/

7.  Edit the file `/media/NIXBOOT/boot/grub/grub.cfg`. Start from an existing grub.cfg, say from your hard drive or from another working bootable USB stick (e.g., from GRML.org). Then, merge in the entry for NixOS from `/media/iso/boot/grub/grub.cfg`, modifying the LABEL to be that of your VFAT partition:
        menuentry "NixOS Installer / Rescue" {
          linux /boot/bzImage init=/nix/store/p94ckcksmhj90cr868cpcajrqgzwy57w-nixos-0.2pre4463_5e88e9b-c877f45/init root=LABEL=NIXBOOT
          initrd /boot/initrd
        }

8.  To share your USB stick with other distributions (e.g., GRML, or both 32- and 64- bit NixOS), you can move NixOS's bzImage and initrd to different location (e.g., `/boot/nix32/` and `/boot/nix64/`), and you appropriately edit the entries in your grub.cfg. If you have both nix32 and nix64, only one can have its `nix-store.squashfs` in the root of any given partition. That's where you can use a second partition, for the second one. For instance, copy the contents of your 64-bit NixOS to the first partition, with label `NIX64`, and copy the nix-store.squashfs of your 32-bit NixOS to the second partition with label `NIX32` (and create an empty nix/store). Copy the 32-bit kernel and initrd to `/boot/nix32/` on the bootable partition, and edit the grub.cfg of the bootable partition accordingly.
9.  There you go: you have a bootable NixOS USB drive, possibly allowing you to boot NixOS 64-bit, NixOS 32-bit, GRML 64-bit, GRML 32-bit, FreeBSD, FreeDOS, boot-sector-tetris, etc. Just you convince your BIOS to let you boot off it.

[Category:Installation](/Category:Installation "wikilink")