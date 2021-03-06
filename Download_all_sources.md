---
title: Download all sources
permalink: /Download_all_sources/
---

Here is a one-liner for downloading all the source dependencies of a package:

nix-store -r $(grep -l outputHash $(nix-store -qR $(nix-instantiate '<nixpkgs>' -A geeqie) | grep '.drv$'))

Here's how it works:

-   $(nix-instantiate '<nixpkgs>' -A geeqie): instantiate geeqie into .drv files and print the file names
-   $(nix-store -qR PREVIOUS | grep '.drv$'): print all references/requirements of the above and keep only the .drv files (which is where static derivations live)
-   $(grep -l outputHash PREVIOUS): keep only the source derivations, since those will have a predefined hash of the output
-   nix-store -r PREVIOUS: realize those derivations, downloading all sources and storing them in the nix store

Phew :-)