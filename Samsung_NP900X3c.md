---
title: Samsung NP900X3c
permalink: /Samsung_NP900X3c/
---

Overview
========

Most of the features seem to be working with Linux &gt;= 3.9

Hardware
--------

-   CPU Intel(R) Core(TM) i5-3317U CPU @ 1.70GHz
-   RAM 4 GB
-   HDD 128GB SSD
-   Screen 13.3-Inch Screen
-   Graphics Intel HD Graphics 4000, Ivy bridge

Configuration
=============

Full configurations is in my [nixpkgs branch](https://raw.github.com/grwlf/nixpkgs/local/machines/samsung-np900x3c-v2.nix). Note, it requires several local packages.

### Touchpad

Touchpad is detected as 'ETPS/2 Elantech Touchpad'. xf86-input-synaptics handles it well. Corresponding config lines:

      services.xserver = {
        synaptics = {
          enable = true;
          accelFactor = "0.05";
          maxSpeed = "10";
          twoFingerScroll = true;
          additionalOptions =
            ''
            MatchProduct "ETPS"
            Option "FingerLow"                 "3"
            Option "FingerHigh"                "5"
            Option "FingerPress"               "30"
            Option "MaxTapTime"                "100"
            Option "MaxDoubleTapTime"          "150"
            Option "FastTaps"                  "1"
            Option "VertTwoFingerScroll"       "1"
            Option "HorizTwoFingerScroll"      "1"
            Option "TrackstickSpeed"           "0"
            Option "LTCornerButton"            "3"
            Option "LBCornerButton"            "2"
            Option "CoastingFriction"          "20"
            '';
          };
      };

### Wireless

System requires iwlwifi-6000g2b-6.ucode in order to work. I've extracted the file from some debian package and placed it into /root/firmware. Corresponding config settings:

      hardware.firmware = [ "/root/firmware" ];

Problems
========

There are some. See

-   [Ubuntu thread](http://ubuntuforums.org/showthread.php?t=1737086)
-   [Kernel.org bug](http://bugzilla.kernel.org/show_bug.cgi?id=44161)
-   [jablonskis.org](http://jablonskis.org/2012/linux-and-samsung-series-laptop-9-fn-keys/)

### BIOS problems

I had to disable SSD boot completely in order to boot from USB. Just changing boot priority didn't help.

Fix: Update BIOS up to recent version

### Battery

Battery charging/discharging indicator doesn't work good.

Fix: Update BIOS up to recent version

### Lid

Acpi thinks lid is always open

    [ierton@greyblade:~]$ cat /proc/acpi/button/lid/LID0/state
    state:      open

Related [Kernel bug \#44161](https://bugzilla.kernel.org/show_bug.cgi?id=44161)

### Multimedia keys

-   rfkill/fanless don't work
-   volume up/down don't work
-   brightness up/down work, but release is broken
-   touchpad disable works

Related discussion on [jablonskis.org](http://jablonskis.org/2012/linux-and-samsung-series-laptop-9-fn-keys/)

### System hangups

I expected runtime system freezes on Linux kernels prior to 3.9. Probably, that was because of bugs in intel video drivers. In 3.9 they are gone:

       boot.kernelPackages = pkgs.linuxPackages_3_12;