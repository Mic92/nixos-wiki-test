---
title: Emacs configuration
permalink: /Emacs_configuration/
---

Tramp
-----

Tramp needs to call \`ls\` and \`id\` but it will not find them on a NixOS machine without additional configuration because they are not in the usual places. To fix this you can add the following to your ~/.emacs (require 'tramp) (add-to-list 'tramp-remote-path "/var/run/current-system/sw/bin")

You can then edit local files as root e.g. C-x C-f /su::/etc/nixos/configuration.nix

Nix mode
--------

(load-library "nix-mode")