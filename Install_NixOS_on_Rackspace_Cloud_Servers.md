---
title: Install NixOS on Rackspace Cloud Servers
permalink: /Install_NixOS_on_Rackspace_Cloud_Servers/
---

This are just a few notes I took down while performing a NixOS installation on a freshly created Rackspace Cloud Server running Gentoo. This is not the ideal way to install NixOS, but it is a way to get things done since bootstrapping NixOS from within a Linux distribution is currently broken.

Prerequisites
-------------

We are going need the contents of the following files, so it might be a good idea to write them down or download those files to your local machine.

`# cat /etc/resolv.conf | tail -n 2`
`nameserver 72.3.128.240`
`nameserver 72.3.128.241`

`# cat /etc/conf.d/net | tail -n 9`
`modules="ifconfig"`
`config_eth0="209.61.142.12 netmask 255.255.255.0"`
`routes_eth0="default via 209.61.142.1"`
`dns_servers_eth0="72.3.128.240`
`72.3.128.241"`
`config_eth1="10.180.133.23 netmask 255.255.128.0"`
`routes_eth1="10.176.0.0/12 via 10.180.128.1`
`10.191.192.0/18 via 10.180.128.1"`

`# cat /etc/fstab | tail -n 4`
`/dev/xvda1  /            ext3     defaults,noatime,barrier=0    1 2`
`/dev/xvdc1  swap         swap     defaults                      0 0`
`proc        /proc        proc     defaults                      0 0`
`shm         /dev/shm     tmpfs    nodev,nosuid,noexec           0 0`

Installation
------------

I am assuming that you are starting from a fresh Gentoo installation. This might not be necessary. Just be aware that this procedure will destroy all data on the server. So have backups ready in case you do have important data on it.

First, you need to put your Cloud Server into Rescue Mode from the Rackspace Cloud control panel. This will mount a rescue system. Your actual system is available at /dev/xvdb1 (root) and /dev/xvdd1 (swap).

First thing we want to do is formatting our root partition and mount it to /mnt.

`# mkfs.ext3 /dev/xvdb1`
`# mount /dev/xvdb1 /mnt`

Next we are going to create a NixOS system from the minimal installation CD. We are going to mount the installation CD to ~/inst and create the NixOS system in ~/host. This system will be used to install NixOS to /mnt. For that we need the unsquashfs binary from squashfs-tools which do not come with the default Gentoo installation. Install it using the following commands.

`# emerge --sync`
`# emerge squashfs-tools`

Now you can download the latest minimal installation CD and create the host system from it.

`# cd ~`
`# mkdir -p inst host/nix/store`
`# wget `[`http://nixos.org/releases/nixos/latest-iso-minimal-x86_64-linux`](http://nixos.org/releases/nixos/latest-iso-minimal-x86_64-linux)
`# modprobe loop`
`# mount -o loop latest-iso-minimal-x86_64-linux inst`
`# unsquashfs -d host/nix/store inst/nix-store.squashfs '*'`

To have a working network connection, copy /etc/resolv.conf to the host/nix/etc. For a proper chroot, you need to bind /dev, /proc and /sys directories.

`# cp /etc/resolv.conf host/nix/etc/`
`# mkdir host/dev host/proc host/sys`
`# for fn in dev proc sys; do mount --bind /$fn host/$fn; done`

To properly chroot into the host system you must locate the packages named nixos and bash. The following commands may prove helpful.

`# find host -type f -name init | grep nixos`
`host/nix/store/abwlkvzyjd2i39b1l1wfv7v9ilx88fwi-nixos-0.1pre34067-34077/init`
`# find host -type f -name bash | tail -n 1`
`host/nix/store/bmgq2jrn6719r6j55gs4rzfp0azcbazy-bash-4.2-p24/bin/bash`

Next we have to edit the host system's init file by removing the upstart task and adding a bash session to it. Since upstart is started in the last line, you might just replace that line with bash.

`# sed '$d' < echo "/nix/store/bmgq2jrn6719r6j55gs4rzfp0azcbazy-bash-4.2-p24/bin/bash" > host/nix/store/abwlkvzyjd2i39b1l1wfv7v9ilx88fwi-nixos-0.1pre34067-34077/init`

Now we are able to chroot into the NixOS system used for installation. Use the following command replacing the store path in case it has changed. Futher commands to be executed in the chrooted environment will be prefixed with "\# (chroot)".

`# chroot host /nix/store/abwlkvzyjd2i39b1l1wfv7v9ilx88fwi-nixos-0.1pre34067-34077/init`

Installation is now straight forward. All you have to do is create a suitable configuration file for booting. Below is a file pasted for reference to get things started. Be sure to replace networking information according to /etc/resolv.conf and /etc/net from the original Cloud Server.

`# (chroot) nixos-generate-config --root /mnt`
`# (chroot) $EDITOR /etc/nixos/configuration.nix`

`{config, pkgs, modulesPath, ...}:`
`{`
`  require = [`
`    "${modulesPath}/virtualisation/xen-domU.nix"`
`  ];`
`  networking = {`
`    useDHCP = false;`
`    defaultGateway = "209.61.142.1";`
`    nameservers = [ "72.3.128.240" "72.3.128.241" ];`
`    interfaces = [`
`      { ipAddress = "209.61.142.12";`
`        name = "eth0";`
`        subnetMask = "255.255.255.0";`
`      }`
`      { ipAddress = "10.180.133.23";`
`        name = "eth1";`
`        subnetMask = "255.255.128.0";`
`      }`
`    ];`
`  };`
`  fileSystems = [`
`    { mountPoint = "/";`
`      device = "/dev/xvda1";`
`      options = "noatime,nodiratime,errors=remount-ro,barrier=0";`
`    }`
`  ];`
`  swapDevices = [`
`    { device = "/dev/xvdc1"; }`
`  ];`
`  services.openssh.enable = true;`
`}`

Before executing nixos-install you might want to run nixos-checkout to obtain the most recent version of packages to be installed into your NixOS system. You might need to copy /etc/resolv.conf from the rescue system to the host system.

`# (chroot) nixos-checkout`
`# (chroot) nixos-install`

Now your system should be set up. Be sure to change the root password to be able to login and double check that /mnt/boot/grub/menu.lst is present and looks sensible.

`# (chroot) passwd`

Now you may exit Rescue Mode from your Rackspace's control panel and have your new NixOS system boot. You may login via SSH using the root account and password specified above.

[Category:Deployment](/Category:Deployment "wikilink")