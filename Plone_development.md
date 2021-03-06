---
title: Plone development
permalink: /Plone_development/
---

How to use Nix to create Plone development environment
------------------------------------------------------

All files can also be downloaded from following gist: <https://gist.github.com/garbas/5293558>

In above gist there is also script called "install_nix.sh" which might help you with setting up Nix.

For rest of the steps It is assumed you already installed Nix on your system.

Quickstart
----------

1. Download gist from link above 2. (optional) In case you don't have Nix install you can use "install_nix.sh" to install it. 3. Run "make" in folder you extracted Gist files 4. Depending on your connection (~5min fast, ~20min slow) Plone 4.3rc1 (at the time of writing this HowTo) will be installed. Start Plone with "bin/instance fg".

What actually happened above?
-----------------------------

Gist from above contains few files:

`- buildout.cfg`
`- plone.nix`
`- Makefile`

"buildout.cfg" is actually a normal buildout configuration file we are used in Plone already. Only thing special about it that we have to pin few eggs to specific versions (look at versions section).

    [buildout]
    extends = http://dist.plone.org/release/4.3rc1/versions.cfg
    eggs-directory = eggs
    versions = versions
    unzip = true
    parts =
    instance

    [instance]
    recipe = plone.recipe.zope2instance
    user = admin:admin
    debug-mode = on
    verbose-security = on
    eggs =
        Pillow
        Plone

    [versions]
    zc.buildout = 1.7.0
    distribute = 0.6.34
    plone.recipe.zope2instance = 4.2.10

"plone.nix" is nix expression where we define which things should be installed for us by nix. Expression below tells to use Python 2.7, Plone and some other eggs which are listed under "paths".

    { }:

    let
        pkgs = import <nixpkgs> { };
    in

    with pkgs;

    buildEnv {
        name = "plone-env";
        paths = [
            python27
            python27Packages.distribute
            python27Packages.recursivePthLoader
            python27Packages.virtualenv
            plone43Packages.plone
            plone43Packages.pillow
            plone43Packages.mailinglogger
            plone43Packages.plone_recipe_zope2instance
            plone43Packages.zc_recipe_egg
        ] ++ lib.attrValues python27.modules;
    }

"Makefile". Convinient place to gather commands but essantially what happends when you run make is:

1. Download (if not already downloaded) all nix packages listed in "plone.nix" and create profile in current folder called "nixenv". On NixOS you don't need to export NIX_PATH.

`   NIX_PATH=~/.nix-defexpr/channels nix-build --out-link nixenv plone.nix`

2. Using virtualenv command which you installed from previous command create python virtual environment.

`   ./nixenv/bin/virtualenv --distribute --clear .`

3. Make sure virtual environment we created in previous step sees all eggs we installed with nix (in step 1).

`   echo ../../../nixenv/lib/python2.7/site-packages > lib/python2.7/site-packages/nixenv.pth`

4. Use this weird easy_install feature that creates binaries if it doesn't exists already even if egg is already installed.

`   ./bin/easy_install -H "" zc.buildout`

5. "bin/buildout" command created in previous step and you use it like you would with any other buildout installation.

`   bin/buildout`

6. "bin/buildout" should create "bin/instance" or whatever you specified in "buildout.cfg" and by running "bin/instance fg" you'll start Plone in foreground mode.

`   bin/instance fg`

Conclusion
----------

Above example should work cross-platform on Darwin(OSX)/Linux/FreeBSD. I(garbas) tested Linux and OSX 10.8.