---
title: WICD
permalink: /WICD/
---

I have a netbook which I plan to connect to my home WPA2-protected wifi network. Here is my experience of setting up wicd and wireless connection.

### Check the hardware

Check that your wireless adapter is working

    [root@pokemon:~]# ifconfig -a | grep -A 5 wlan
    wlan0     Link encap:Ethernet  HWaddr 00:16:EB:19:DC:A0
              inet addr:192.168.1.100  Bcast:192.168.1.255  Mask:255.255.255.0
              inet6 addr: fe80::216:ebff:fe19:dca0/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:21265 errors:0 dropped:0 overruns:0 frame:0
              TX packets:18615 errors:0 dropped:0 overruns:0 carrier:0

If not, consult dmesg. One possible reason is missing firmware. Some firmware types should be enabled explicitly, see networking.enableIntel2100BGFirmware option as example.

### Disable all NixOS networking, enable wicd

Wicd is fully-function network manager, it takes responsibility for connecting, launching dhcp clients, storing passwords, etc. To prevent conflicts, disable other network agents

    networking = {
      ...
      interfaceMonitor.enable = false;
      enableWLAN = false; # Don't run wpa_supplicant (wicd will do it when necessary)
      useDHCP = false; # Don't run dhclient on wlan0
      wicd.enable = true;
      ...
    };

### Setup wicd

Now rebuild nixos, switch to new configuration and launch wicd gui.

    [root@pokemon:~]# wicd-gtk --no-tray

If it doesn't see wireless networks, open preferences and check wireless interface name. Enter correct value (like "wlan0") if needed.

That's it. Now use gui to find your network, enter passwords and connect to it.