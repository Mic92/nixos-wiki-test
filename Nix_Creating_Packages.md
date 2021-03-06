---
title: Nix Creating Packages
permalink: /Nix_Creating_Packages/
---

Quick Guide
-----------

What you need to quickly test building packages is already on the system. There are various ways to approach this, but for brevity, ease of use and educational purposes we will use nixos-rebuild to test our packages. This guide will teach you who set up a quick testing environment, using the current (14.04) NixOS channel as a source for the scaffolding.

**1. Open a terminal**


Press `(KeyCmb: ALT+F2, write "'''konsole'''" and press ''enter'')`

'''2. Setup a directory or folder for *"nixpkgs"*.


Let's put it inside a folder called *"Builds"* in your *[Home Folder](/NixOS_User_Reference "wikilink")*.'''


**In Dolphin**

Open Dolphin `(KeyCmb: ALT+F2, write "'''Dolphin'''" and press enter)` and select the home directory. Create a Builds folder and create the nixpkgs folder inside of that folder

------------------------------------------------------------------------



***In Terminal***

`mkdir -p /home/$USER/Builds/nixpkgs`

------------------------------------------------------------------------

'''2. Find the nixpkgs channel stored in NixOS


Switch to an open terminal


**Echo the NIX_PATH**

Simply run `echo $NIX_PATH`, which should output something like this:


`/nix/var/nix/profiles/per-user/root/channels/nixos:nixpkgs=/etc/nixos/nixpkgs:nixos-config=/etc/nixos/configuration.nix`

Here we can see that `nixpkgs=/etc/nixos/nixpkgs` is a virtual directory (or symlinked directory) that leads to the current active set of channels that provides the various versions and iterations of `nixpkgs`.

Quick Guide Notes
-----------------

Re: **mkdir -p**

> *mkdir stands for "make directory" and "-p" stands for --parents, visa vi parent folders or directories*

To do: Add content for this page: "Creating Nix Packages"

Standard Environment
--------------------

Most packages build their software in the "standard environment". Read the [Standard Environment wiki documentation](/NixPkgs_Standard_Environment "wikilink").