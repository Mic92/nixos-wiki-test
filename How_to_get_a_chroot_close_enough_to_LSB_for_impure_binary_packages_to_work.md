---
title: How to get a chroot close enough to LSB for impure binary packages to work
permalink: /How_to_get_a_chroot_close_enough_to_LSB_for_impure_binary_packages_to_work/
---

One of the options is special-chroot script by Michael Raskin.

Checkout configurations/ tree from Nix SVN repository. The exact directory is <https://nixos.org/repos/nix/configurations/trunk/misc/raskin/misc-scripts> - the file is special-chroot. It expects to be run on NixOS, although I have used it on non-NixOS Nix installations.

It has the following parameters:

1) Root path where to prepare all the stuff 2) Home path as seen after chroot 3) User name 4) '' or 'usr-only' : if usr-only, /bin /sbin /lib are skipped 5) '' or path : what to use instead of /var/run/current-system/sw

Tries to make most installers safe by mounting most things ro. /old-root is mounted as rw, though.

Tries to unmount everything on termination.