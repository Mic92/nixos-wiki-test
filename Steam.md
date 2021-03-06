---
title: Steam
permalink: /Steam/
---

This page is intended to describe the current state of Steam under Nixos and to discuss what the problems are in packaging it and how we can approach solving them.

Description of the package
--------------------------

Steam is distributed as a .deb file, for now only as an i686 package (the amd64 package only has documentation). When unpacked, it has a script called steam that in ubuntu (their target distro) would go to /usr/bin. When run for the first time, this script copies some files to the user's home, which include another script that is the ultimate responsible for launching the steam binary, which is also in $HOME.

Nix problems and constraints
----------------------------

-   We don't have /bin/bash and many scripts point there
-   We don't have the dinamic loader in /lib
-   The steam.sh script in $HOME could not be patched last I tried, as it is checked and rewritten by steam
-   The steam binary cannot be patched, it's also checked

Approaches
----------

### Link bash to /bin and glibc/lib to /lib and be happy

-   Pros: easy, works
-   Cons: not very nix-compliant

### Workaround the scripts and launch steam directly

-   Pros: not so hard
-   Cons: this only solves the part concerned with running steam. What about the games? We can patch some of them, but at least Team Fortress is checked and rewritten if modified

This is the approach that I (page) took in my github branch: <https://github.com/cpages/nixpkgs/tree/steam>

### Intercept Steam's calls with LD_PRELOAD or the like

-   Pros: more robust
-   Cons: difficult to achieve and may be broken by changes in the binary

aszlig started working in this in his branch: <https://github.com/aszlig/nixpkgs/tree/steam>

### chroot

-   Pros: it would allow us to have binaries in the expected paths without disrupting the system
-   Cons: performance?

shlevy/aristid proposed this approach. Noone has tried :)

I want to play
--------------

Ok, now for the fun part. This is the procedure that should allow you to play:

1.  Checkout <https://github.com/cpages/nixpkgs/tree/steam>
2.  Install steam (nix-env -iA steam -f path/to/repo). Might take some time
3.  Launch steam: the first run will install stuff to your home, and end up failing with wrong interpreter bla bla bla...
4.  Launch steam again: if all went well, you should end up with steam auto-updating itself. When it finishes, it won't automatically start
5.  Launch steam once more: this should present you with the login screen, and from here to a working steam

### But what about the games?

You can install any of the games normally, but they will fail to start. From this step on, you're in the unsupported realm. Some games can be patchelfed:

` patchelf --set-interpreter /path/to/ld.so game_binary (you can get the path to an x86 ld.so looking at the steam script in the store, for me /nix/store/xh0q23rgqbjfrh3zfv4jyxvcvjnxqh64-glibc-2.15.0/lib/ld-linux.so.2)`

You might also need patching some scripts. That all depends on each one. You can then press play from steam and if you're lucky that'll be it!

Troubleshooting
---------------

I was sure you would reach this part.

### Steam fails to start. What do I do?

strace is your friend.

### Game X fails to start

Games may fail to start because they lack dependencies (this should be added to the script, for now), or because they cannot be patched. The steps to launch a game directly are:

1.  Patch the script/binary if you can
2.  Add a file named steam_appid.txt in the binary folder, with the appid as contents (it can be found in the stdout from steam)
3.  Using the LD_LIBRARY_PATH from the nix/store steam script, with some additions, launch the game binary

` LD_LIBRARY_PATH=~/.steam/bin32:$LD_LIBRARY_PATH:/nix/store/pfsa... blabla ...curl-7.29.0/lib:. ./Osmos.bin32 (if you could not patchelf the game, call ld.so directly with the binary as param)`

With this technique, I can play many games directly from steam. Others, like Team Fortress, cannot be patched so I only managed to run them from the cmd line.