---
title: Wpa supplicant
permalink: /Wpa_supplicant/
---

Wireless networking
===================

NixOS' uses wpa_supplicant for wireless networking. This article describes how to configure NixOS to use wpa_supplicant and to connect to a variety of wireless networks.

Configuration
-------------

To enable wireless networking add the following attribute to your `configuration.nix` and reconfigure your system.

`networking.enableWLAN = true;`

Prerequisites
-------------

Before proceeding make sure to check wheather your wireless network card has properly been recognized. You can check this by issuing `iwconfig` and you should see a wireless extension being reported for `wlan0`.

Wirless networks
----------------

Configuration of wireless networks is being done in `/etc/wpa_supplicant.conf`. This file is readable and writeable by root only so make sure to have root privileges before editing. Since this file contains plain passwords for wireless networks you shouldn't attempt to make this file readable by any other account than root.

The file `wpa_supplicant.conf` contains a lot of configuration and documentation already. However, these are sensible default values which should work in most cases. As of now don't thouch them unless something isn't working as expected or you know what you are doing.

To the end of the file you can describe the wireless networks you want to connect to. In most cases you only need to specify a essid and a password. To add a wireless network callsed `mynetwork` with passphrase `mysecretpass` add the following contents to the end of the file `/etc/wpa_supplicant.conf`.

`network={`
`  ssid="mynetwork"`
`  psk="mysecretpassphrase"`
`}`

You now should reconfigure your system again and wpa_supplicant should take care of automatically getting you connected as soon as you are in range for any of the networks specified in `/etc/wpa_supplicant.conf`.