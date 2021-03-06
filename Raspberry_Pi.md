---
title: Raspberry Pi
permalink: /Raspberry_Pi/
---

Current State
-------------

**NixOS:** The compatibility of Raspberry Pi and the latest version of NixOS is unknown, but the disk image linked on this page will successfully install, boot, and run NixOS.

**NixPkgs:** Can bootstrap an armhf NixPkgs for the Raspberry Pi. (Patched with the [0236cc5d8826](https://github.com/NixOS/nixpkgs/commit/0236cc5d8826) commit.)

To read development history, see the following pages:

-   [Add Raspberry-Pi support to NixOS](https://github.com/NixOS/nixos/issues/66) -
-   [Add Raspberry-Pi stdenvLinux support](https://github.com/NixOS/nixpkgs/issues/234) (pure nixpkgs)

Installation of NixOS Image
---------------------------

### Download Image

To download the image file, open the following magnet link in a BitTorrent client, such as Transmission.

`   `[`magnet:?xt=urn:btih:3a1273137a31ef65453ddc44eafcf8e0ede24731&dn=nixos-raspberrypi-ef28e8e70e96-0236cc5d8826-sd2g.img.xz`](magnet:?xt=urn:btih:3a1273137a31ef65453ddc44eafcf8e0ede24731&dn=nixos-raspberrypi-ef28e8e70e96-0236cc5d8826-sd2g.img.xz)

### Write Image to SD Card

`   df -l`
`   # Insert SD card into computer`
`   df -l`
`   # Note the name of the newly added device. e.g. sdb1, sdc1`

`   # Assuming the device name of the SD card is "/dev/sdb1"`
`   sudo apt-get install dcfldd`
`   sudo umount /dev/sdb1`
`   # Double-check the following command. It's powerful!`
`   sudo dcfldd bs=1M if=/home/alex/Downloads/nixos-raspberrypi-ef28e8e70e96-0236cc5d8826-sd2g.img of=/dev/sdb`
`   # Wait for writing to complete.`

### Expand the Partition

The size of the NixOS partition is only 2 GB, but your SD card might be larger. To expand this to fill the SD card and add a small swap partition, use the `gparted` program as instructed by [this elinux.com wiki page](http://elinux.org/RPi_Resize_Flash_Partitions#Manually_resizing_the_SD_card_using_a_GUI_on_Linux).

### Logging In

If you have problems logging in, read the "Set a Password for Root" section of [this blog post](https://bitbucket.org/chexxor/researchnotes/src/ffc4c9622154aec847f3091d74e150a0a9d336cb/CodeArticles/InstallingNixOsOnRaspberryPi.md?at=master) for possible solutions.

Improving Build Speed
---------------------

### Binary Cache

The Raspberry Pi compiles things **very** slowly. As Nix compiles everything from source, this is a problem for Nix on Raspberry Pi. This problem can be alleviated by setting up Nix to query a shared binary-cache, but this currently does not exist for this specific platform. The community should figure out how to create a binary-cache.

### Distributed Builds

Another way to speed up builds is to share the computation with other machines. One tool which does this is called [`distcc`](https://code.google.com/p/distcc/).

The following notes were taken when first implementing `distcc` for NixOS on Raspberry Pi.

#### Implementing distcc

##### On Raspberry Pi

How do we inject `distcc` into the Nix build system? We can use a new feature of stdenv-updates stdenv, called `userHook`. Nix wants to avoid changing the compiler, so by using distcc to compile, we are introducing an "impurity". To introduce this impurity into our NixPkgs, we can call a shell script like this:

`{`
`  stdenv.userHook = "if [ -f /niximpure/impure.sh ]; then . /niximpure/impure.sh; fi";`
`}`

If you build NixOS, remember to set the `userHook` in the `nixpkgs.config` option.

The `impure.sh` script will load `distcc`. Here is a simplification:

`case $out in`
`  *)`
`     case $NIX_GCC in`
`       *-gcc-wrapper-4.7* | *-bootstrap-gcc-wrapper*)`
`         echo Using distcc for gcc 4.7`
`         export DISTCC_HOSTS=192.168.1.35`
`         PATH=/niximpure/distcc:$PATH`
`         export HOME=/niximpure/home`
`         ;;`
`       *-gcc-wrapper-4.6*)`
`         echo Using distcc for gcc 4.6`
`         export DISTCC_HOSTS=192.168.1.35:3633`
`         PATH=/niximpure/distcc:$PATH`
`         export HOME=/niximpure/home`
`         ;;`
`     esac`
`   ;;`
`esac`

The `/niximpure` tree which is referenced looks like this.

`# ls -lR /niximpure`
`/niximpure:`
`total 16`
`drwxr-xr-x 2 root root 4096 28 des 00:06 distcc`
`drwxrwxr-x 4 root 1002 4096  4 gen 14:32 home`
`-rw-r--r-- 1 root root 1351 26 gen 13:06 impure.sh`
`drwxr-xr-x 3 root root 4096 29 des 18:57 pump`
`/niximpure/distcc:`
`total 0`
`lrwxrwxrwx 1 root root 15 23 des 21:09 c++ -> /usr/bin/distcc`
`lrwxrwxrwx 1 root root 15 23 des 21:08 cc -> /usr/bin/distcc`
`lrwxrwxrwx 1 root root 15 23 des 21:08 g++ -> /usr/bin/distcc`
`lrwxrwxrwx 1 root root 15 23 des 21:08 gcc -> /usr/bin/distcc`
`/niximpure/home:`
`total 0`

Notice the `/usr/bin/distcc` directory, which allows us to point at any `distcc` version you have installed, such as one in `/usr` or one which Nix built.

When building, rather than using `/usr/bin/gcc`, the `gcc` found in the `PATH` environment variable will be used.

##### On Shared Machines

Having a copy of NixPkgs, you can build your cross-compilers for `distcc`. Prepare a directory for them:

`mkdir distcc`
`cd distcc`

And create a Nix file, such as `pi.nix`, with these contents:

`let`
`  pkgsFun = import `<nixpkgs>`;`
`  pkgsNoParams = pkgsFun {};`
`in`
`pkgsFun`
`{`
`   crossSystem = {`
`       config = "armv6l-unknown-linux-gnueabi";`
`       bigEndian = false;`
`       arch = "arm";`
`       float = "hard";`
`       fpu = "vfp";`
`       withTLS = true;`
`       libc = "glibc";`
`       platform = pkgsNoParams.platforms.raspberrypi;`
`       openssl.system = "linux-generic32";`
`       gcc = {`
`         arch = "armv6";`
`         fpu = "vfp";`
`         float = "softfp";`
`         abi = "aapcs-linux";`
`       };`
`   };`
`   config = pkgs: {`
`     packageOverrides = pkgs : {`
`       distccMasquerade = pkgs.distccMasquerade.override {`
`         gccRaw = pkgs.gccCrossStageFinal.gcc;`
`         binutils = pkgs.binutilsCross;`
`       };`
`     };`
`   };`
`}`

In case `gcc_realCross = gcc47_realCross`, you can build the `gcc 4.7` cross-compiler this way. (The `gcc 4.7` compiler is the compiler in the bootstrap-tools to build the stdenv.

`nix-build -o pi-gcc47 distccMasquerade pi.nix`

You can then start distcc serving this, having distcc installed, and running:

`` PATH=`pwd`/pi-gcc47/bin:$PATH distccd --allow 192.168.1.0/24 --stats --stats-port 8032 ``

Once stdenv is built, as it includes gcc 4.6, you will need the cross-compiler gcc 4.6, which you can build using the same technique as before, but changing `gcc_realCross = gcc46_realCross` in nixpkgs' `all-packages.nix`.

Notes
-----

NixOS at [viric/master](https://github.com/viric/nixos/tree/master) can prepare a proper bootloader (vfat at `/boot`).

Having the SD vfat that the dumb loader will read at `/boot`, here is an example `configuration.nix` file:

`{pkgs, config, ...}:`
`{`
`  boot.loader.grub.enable = false;`
`  boot.loader.generationsDir.enable = false;`
`  boot.loader.raspberryPi.enable = true;`
`  boot.kernelPackages = pkgs.linuxPackages_3_6_rpi;`
`  boot.kernelParams = [`
`    "coherent_pool=6M"`
`    "smsc95xx.turbo_mode=N"`
`    "dwc_otg.lpm_enable=0"`
`    "root=/dev/sda5"`
`    "rootwait"`
`    "console=tty1"`
`    "elevator=deadline"`
`  ];`
`  # cpufrequtils doesn't build on ARM`
`  powerManagement = false;`
`  fileSystems = [`
`    {`
`      device = "/dev/mmcblk0p1";`
`      mountPoint = "/boot";`
`      fsType = "vfat";`
`    }`
`    {`
`      device = "/dev/sda5";`
`      mountPoint = "/";`
`      fsType = "ext4";`
`    }`
`  ];`
`  services.xserver.enable = false;`
`  services.openssh = {`
`    enable = true;`
`    permitRootLogin = "yes";`
`  }; `
`}`