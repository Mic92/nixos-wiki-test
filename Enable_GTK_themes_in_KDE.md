---
title: Enable GTK themes in KDE
permalink: /Enable_GTK_themes_in_KDE/
---

XFCE gtk themes are already available in NixOS so a simple way to make them available is to add the following to your configuration.nix:

    services.xserver.desktopManager.xfce.enable = true;

KDE will run any scripts you place in ~/.kde/env on startup so this is a good place to set the GTK_PATH so that applications can find the theme engine. If the path is wrong/unset you will see errors like this:

    Gtk-WARNING **: Unable to locate theme engine in module_path

~/.kde/env/set-gtk-theme.sh

    #!/bin/sh

    export GTK_PATH=/var/run/current-system/sw/lib/gtk-2.0
    export GTK2_RC_FILES=/var/run/current/sw/share/themes/Xfce-curve/gtk-2.0/gtkrc