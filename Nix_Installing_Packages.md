---
title: Nix Installing Packages
permalink: /Nix_Installing_Packages/
---

How to install a software package using Nix?
--------------------------------------------

The easiest way to get Nix expressions is to use the community's NixPkgs repository. After setting it up, use the `nix-env` command to install the packages defined there.

`   $ nix-channel --add `[`http://nixos.org/channels/nixpkgs-unstable`](http://nixos.org/channels/nixpkgs-unstable)
`   $ nix-channel --update`
`   $ nix-env --install hello`

The [`nix-channel`](http://nixos.org/nix/manual/#sec-nix-channel) command registers URL which has a Nix expression. Then, we use `nix-channel --update` to download this Nix expression. Finally, we use the `nix-env --install` command to install the "hello" package. The [`nix-env`](http://nixos.org/nix/manual/#sec-nix-env) command installs or removes software in our environment.

Besides the "hello" package, what else can we install? For more options, see the other attributes in the [`all-packages.nix`](https://github.com/NixOS/nixpkgs/blob/master/pkgs/top-level/all-packages.nix#L8107) file. Most of these attributes, those which are defined by the `callPackage` or `mkDerivation` functions, creates a "Nix package". These functions produce a package by being passed a "Nix expression" which defines the package. These Nix expressions are stored elsewhere in the `pkgs` directory to keep the package list easy to read.

Let's install some other packages we found in the `all-packages.nix` file.

`   $ nix-env --install firefox`
`   $ nix-env --install dropbox bitcoin`
`   $ nix-env -i dropbox bitcoin`

How to query available software packages?
-----------------------------------------

Rather than looking at the `all-packages.nix` file to find packages, we can also use the `nix-env --query --available` command on the command line.

`   $ nix-env --query --available firefox`
`   $ nix-env --query --available`
`   $ nix-env --query --available | grep firefox`
`   $ nix-env -q -a | grep firefox`
`   $ nix-env -qa | grep firefox`

We can also see which packages we currently have installed by using the `nix-env --query --installed` command.

`   $ nix-env --query --installed`
`   $ nix-env -q`

How to uninstall software packages?
-----------------------------------

And of course, we can uninstall packages by using the `nix-env --uninstall` command.

`   $ nix-env --uninstall firefox`
`   $ nix-env -e firefox`

Further reading
---------------

More information about how Nix works can be found from the [Nix Main Page](/Main_Page_Package_Manager "wikilink").