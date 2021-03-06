---
title: Multiplatform NixOS
permalink: /Multiplatform_NixOS/
---

Overview
========

The current supported platforms are:

-   PC (i686-linux, x86_64-linux)
-   Sheevaplug (armv5tel-linux)
-   Fuloong Mini-PC (mips64, with mips n32 binaries) - Only in stdenv-updastes

Fuloong Mini-PC
===============

Installation
------------

I will describe the procedure to install NixOS through the network, although a bootable USB could work.

Requirements:

-   A TFTP server
-   An NFS server
-   Download the [system tarball](http://vicerveza.homeunix.net/~viric/tmp/nix/nixos-system-mips64-linux.tar.xz) and unpack it somewhere accessible by the NFS client you will have in your Fuloong Mini-PC. You can use an 'exports' like:

`/home/fuloongroot 192.168.1.0/24(rw,no_root_squash,no_all_squash)`

Procedure:

In the PC, lets prepare the files you will serve by NFS

-   Read the `kernel-parameters.txt`. Notice there is a mention to an `init-stage2.sh` file and a `system` directory, both in the store path.
-   Make symbolic links to those paths in /boot/init and /boot/system. Something like this:

`ln -s /nix/store/41x8chz5mwf3nwxd7460skz291fyvml7-stage-2-init.sh boot/init`
`ln -s /nix/store/rmc2a3w0c10rwhad7493scqcrh5b0dkc-system boot/system`

-   Disable the start of dhclient, editing nix/store/\*upstart-dhclient.conf and commenting the line starting with "start on"
-   Copy the file boot/vmlinux to your TFTP server root

In the Fuloong2f:

-   Boot PMON (turn on the computer), and enter its command line mode using a USB keyboard and pressing at boot time the DELETE key. You will get a prompt.
-   Give your system an IP (use 'devls' to know about your NIC device name)

`ifaddr rtk0 192.168.1.223`

-   Get the kernel through tftp, where TFTPIP is your tftp server ip

`load `[`tftp://TFTPIP/vmlinux`](tftp://TFTPIP/vmlinux)

-   Boot the kernel, where NFSIP:NFSPATH is your nfsroot mount point, and IP is the ip of your computer

`g console=tty nfsroot=NFSIP:NFSPATH root=/dev/nfs ip=IP init=/boot/init systemConfig=/boot/system`

After this, you will have a running NixOS. Nevertheless, here are some details you will still need:

In order to use nix-env in that nfsroot, you will need to setup your ~/.nixpkgs/config.nix file to set the platform you want the packages for:

`pkgs:`
`{`
`  platform = pkgs.platforms.fuloong2f_n32;`
`}`

Additionally, you will need to checkout the following *nixos* and *nixpkgs* to work properly in the fuloong:

`cd /etc/nixos`
`svn co `[`https://svn.nixos.org/repos/nix/nixos/branches/stdenv-updates`](https://svn.nixos.org/repos/nix/nixos/branches/stdenv-updates)` nixos`
`svn co `[`https://svn.nixos.org/repos/nix/nixpkgs/branches/stdenv-updates`](https://svn.nixos.org/repos/nix/nixpkgs/branches/stdenv-updates)` nixpkgs`

After this you can proceed installing programs or installing NixOS with the usual `nixos-install` procedures as described in the [NixOS manual](http://hydra.nixos.org/job/nixos/trunk/manual/latest/download) installing from a bootable CD.

Configuration details
---------------------

In your configuration.nix, you have to pay attention to these basics to get a bootable system. We are going to use the 'generationsDir' boot method, that simply puts the necessary files to load into known paths. You will have to configure your usual PMON /boot/boot.cfg pointing to the proper config.

I will type the attributes involved in the shortest form. If I write boot.loader, it does not mean 'loader' should be the only attribute of boot. It's up to your configuration.

Switch on the generationsDir bootloader.

`boot.loader = {`
`  grub.enable = false;`
`  generationsDir = {`
`    enable = true;`
`    copyKernels = true;  # if you don't want store path references, but references to files in /boot`
`  };`
`};`

Use a recent kernel:

`boot.kernelPackages = pkgs.linuxPackages_2_6_35;`

At the time of writing this, the kernel provides /dev/hd\* devices for the hard disk, while we don't have udev rules to create them. So, I you will need to create the devices at initrd time:

     boot.initrd.postDeviceCommands = ''
       mknod /dev/hda1 b 3 1
       mknod /dev/hda2 b 3 2
     '';

You have to tell nixpkgs that you have a fuloong mini-pc system. So you need:

`nixpkgs.config.platform = pkgs.platforms.fuloong2f_n32;`

You will also need that nixpkgs config option in your user nixpkgs configuration, like ~/.nixpkgs/config.nix, to get nix-env and such commands working fine:

`pkgs:`
`{`
`  platform = pkgs.platforms.fuloong2f_n32;`
`}`

Support
-------

Feel free to ask in the mailing list nix-dev@cs.uu.nl or in irc.freenode.net\#nixos

Sheevaplug
==========

Sheevaplug configuration
------------------------

` boot = {`
`   initrd = {`
`     enableSplashScreen = false;`
`     kernelModules = [ ];`
`     lvm = false;`
`   };`
`   kernelParams = [];  # These are hardcoded in uboot ! So not handled by nixos.`
`   kernelModules = [ "fuse" ];`
`   kernelPackages = pkgs.kernelPackages_2_6_32;`
`   loader = {`
`     grub = {`
`       enable = false;`
`     };`
`     generationsDir = {`
`       enable = true;`
`       copyKernels = true;`
`     };`
`   };`
` };`
` services = {`
`   nixosManual = {`
`     enable = false;`
`   };`
`   mingetty = {`
`     ttys = [ "ttyS0" ];`
`   };`
` };`
` sound.enable = false;`
` fileSystems = [`
`   { mountPoint = "/";`
`     device = "/dev/mmcblk0p2";`
`     options = "relatime";`
`     }`
`   { mountPoint = "/boot";`
`     device = "/dev/mmcblk0p1";`
`     neededForBoot = true;`
`     }`
`   { mountPoint = "/var/log";`
`     device = "tmpfs";`
`     fsType = "tmpfs";`
`     options = "size=128m";`
`     neededForBoot = true;`
`     }`
`   { mountPoint = "/var/run";`
`     device = "tmpfs";`
`     fsType = "tmpfs";`
`     options = "size=32m";`
`     neededForBoot = true;`
`     }`
`   { mountPoint = "/tmp";`
`     device = "tmpfs";`
`     fsType = "tmpfs";`
`     options = "size=128m";`
`     }`
`   { mountPoint = "/nix/var/log";`
`     device = "tmpfs";`
`     fsType = "tmpfs";`
`     options = "size=32m";`
`     }`
`   { mountPoint = "/mnt/hd";`
`     device = "/dev/sda1";`
`     options = "relatime";`
`     }`
`   { mountPoint = "/var/postfix";`
`     device = "/mnt/hd/var/postfix";`
`     options = "bind";`
`     }`
`   { mountPoint = "/var/spool/mail";`
`     device = "/mnt/hd/var/spool/mail";`
`     options = "bind";`
`     }`
`   { mountPoint = "/var/mysql";`
`     device = "/mnt/hd/var/mysql";`
`     options = "bind";`
`     }`
`   ];`
` nixpkgs = {`
`   config = {`
`     php = {`
`       mysqli = false;`
`       pdo_mysqli = false;`
`       bcmath = false;`
`       postgresql = false;`
`       sqlite = false;`
`       soap = false;`
`       gd = true;`
`     };`
`     git = {`
`       svnSupport = true;`
`       guiSupport = true;`
`     };`
`     subversion = {`
`       perlBindings = true;`
`     };`
`     packageOverrides = pkgs: with pkgs; rec {`
`       platform = platforms.sheevaplug;`
`     };`
`   };`
` };`