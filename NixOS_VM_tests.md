---
title: NixOS VM tests
permalink: /NixOS_VM_tests/
---

The NixOS source tree contains tests that use automatically instantiated virtual machines to test various features of the system. These tests define virtual networks of virtual NixOS machines and test scripts that use them. This approach is described in an [ISSRE-2010 paper](http://www.st.ewi.tudelft.nl/~dolstra/pubs/decvms-issre2010-final.pdf). While it uses NixOS in the VMs, it is not limited to testing NixOS: if you have a (distributed) application or system-level package that you wish to test automatically, you can use the NixOS VM testing framework. Note that NixOS is <em>not</em> required on the host system - any Linux system (such as Ubuntu) with KVM support should work.

Running a test
--------------

If you don't have NixOS as your host OS, you should first [install the Nix package manager](http://nixos.org/nix/download.html) and obtain the Nixpkgs and NixOS source trees:

    $ svn co https://svn.nixos.org/repos/nix/nixpkgs/trunk nixpkgs
    $ svn co https://svn.nixos.org/repos/nix/nixos/trunk nixos

To speed up builds by using pre-built binaries, you should do:

    $ nix-pull http://nixos.org/releases/nixpkgs/channels/nixpkgs-unstable/MANIFEST

Now you can run a test as follows:

    $ nix-build nixos/tests/ -A simple.test

This trivial "test" (defined in `nixos/tests/simple.nix`) simply starts a VM and shuts it down.

Writing tests
-------------

TODO