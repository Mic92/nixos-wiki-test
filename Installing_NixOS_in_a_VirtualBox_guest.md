---
title: Installing NixOS in a VirtualBox guest
permalink: /Installing_NixOS_in_a_VirtualBox_guest/
---

Installing NixOS On VirtualBox
------------------------------

Using NixOS with VirtualBox on OS X.

-   Download the NixOs iso for 32 bit from <http://nixos.org>
-   Add a New Machine in VirtualBox with OS Type "Ubuntu"
-   Base Memory Size: 768 MB (or more)
-   New Hard Disk of 8 GB
-   Mount the CD-Rom with the NixOs iso (by clicking on CD/DVD-ROM)
-   Click on Settings / Advanced and enable PAE/NX
-   Networking: select "Attached to: NAT". Change host-interface to "en1: Airport".
-   Now you can install nix using the nixos manual:

` $ fdisk /dev/sda (create a full partition, don't skip this step)`
` $ mke2fs -j -L nixos /dev/sda1 (idem)`
` $ mount LABEL=nixos /mnt`
` $ mkdir -p /mnt/etc/nixos`
` $ nixos-hardware-scan > /mnt/etc/nixos/configuration.nix`
` $ nano /mnt/etc/nixos/configuration.nix`

(add the fileSystems. this is what my final config looks like:)

` {`
`   boot.initrd.extraKernelModules = [  "ata_piix" ];`
`   boot.kernelModules = [  ];`
`   boot.grubDevice = "/dev/sda";`
`   fileSystems = [`
`     { mountPoint = "/";`
`       label = "nixos";`
`     }`
`   ];`
`   nix.maxJobs = 1;`
`   services = {`
`     sshd = { enable = true; };`
`     xserver.videoDriver = "vesa";`
`   };`
` }`

Now we're ready to install!

` $ nixos-install`
` $ reboot`

Now you got a working nixos system. However, you can't conveniently log in to ssh this way, so you'll need to set up port-forwarding. Doing that is explained on this site, all you have to do is create a script named "vbox-tunnel" that contains the following lines:

` #!/bin/bash`
` cd /Applications/VirtualBox.app/Contents/MacOS/`
` ./VBoxManage setextradata "$1" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/$2/Protocol" TCP`
` ./VBoxManage setextradata "$1" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/$2/GuestPort" $3`
` ./VBoxManage setextradata "$1" "VBoxInternal/Devices/pcnet/0/LUN#0/Config/$2/HostPort" $4`

Now execute the script:

` vbox-tunnel Nix sshtunnel 22 2222`

You should be able to log in to Nix now by running `ssh -p2222 localhost` in your OS X Terminal. Enjoy!