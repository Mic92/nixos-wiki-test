---
title: Firefox
permalink: /Firefox/
---

This wiki page contains information about using the Java plugin in Firefox on NixOS.

From Firefox version 3.6, the old Java plugin located in <JRE>/plugin/i386/javaplugin-oji.so no longer works (source: [1](http://www.oracle.com/technetwork/java/javase/manual-plugin-install-linux-136395.html)). The new plugin is located in <JRE>/lib/i386/libnpjp2.so.

For Firefox to reach the plugins, there are 3 ways:

1.  put the path of the plugin in the environment variable MOZ_PLUGIN_PATH
2.  create a symbolic link to the plugin in ~/.mozilla/plugins
3.  create a symbolic link to the plugin in <FIREFOX-DIRECTORY>/plugins

Help to do each method:

1.  The MOZ_PLUGIN_PATH way is implemented in NixOS (look at /etc/nixos/nixpkgs/pkgs/development/compilers/jdk/jdk6-linux.nix) but the path given there is the path to the old plugin whcih does not work. You can correct this by changing the ***if installjdk then "/jre/plugin/i386/ns7" else "/plugin/i386/ns7";*** line to ***if installjdk then "/jre/lib/i386" else "/lib/i386";***'. This works but you will see a lot of warnings in the command line when running Firefox because it tries to load all the libraries in <JRE>/lib/i386 as a plugin. You should also install firefoxWrapper (or firefox-3.6.8-with-plugins if you are using nix-env) instead of firefox and set nixpkgs.config.firefox.jre = true in your /etc/nixos/configuration.nix.
2.  This is an easy way but you should do the same for all users: ***mkdir -p ~/.mozilla/plugins; ls -l \`which java\` | sed 's/.\*-&gt; \(.*\)\\/bin\\/java/ln -s \\1\\/lib\\/i386\\/libnpjp2.so ~\\/.mozilla\\/plugins/' | bash***
3.  The best would be to implement this centrally in /etc/nixos/nixpkgs/pkgs/top-level/all-packages.nix and reach all the firefox plugins this way.
