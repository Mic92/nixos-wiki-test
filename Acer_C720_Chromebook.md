---
title: Acer C720 Chromebook
permalink: /Acer_C720_Chromebook/
---

Overview
========

-   Processor: Intel Celeron 2955U @ 1.40GHz
-   Network controller: Qualcomm Atheros AR9462

Configuration
=============

The C720 comes with a Cypress APA touchpad. Support for this touchpad was added in kernel 3.17-rc1. In order to get the touchpad working CONFIG_CHROME_PLATFORMS needs to be enabled.

For example, for touchpad support with the 3.18 kernel, the following should be added to /etc/nixos/configuration.nix:

` boot.kernelPackages = pkgs.linuxPackages_3_18;`
` nixpkgs.config.packageOverrides = pkgs:`
`   { linux_3_18 = pkgs.linux_3_18.override {`
`       extraConfig =`
`         `**`'`**
`           CHROME_PLATFORMS y`
`         `**`'`**`;`
`     };`
`   };`

[Category:Installation](/Category:Installation "wikilink") [Category:Hardware](/Category:Hardware "wikilink") [Category:Laptops](/Category:Laptops "wikilink")