---
title: Mercurial and Nix repositories
permalink: /Mercurial_and_Nix_repositories/
---

Mercurial and nixos/nixpkgs
---------------------------

You are less than 30 years old and really can't get into non-distributed VCS? Then this page might be what you're looking for!

### Installing Mercurial and hgsubversion

` nix-env -i mercurial`

` nix-env -i hgsubversion`

(hgsubversion will be hopefully soon available via nix...)

### Configuring Mercurial to use hgsubversion

-   EDIT ~/.hgrc

` [ui]`
` username = Firstname Lastname `<email>
` verbose = True`
` [extensions]`
` hgext.convert=`
` rebase=`
` svn=~/.nix-profile/lib/python2.6/hgsubversion`

-   SAVE and QUIT (only the extensions part is important here)
-   check that everuthing is ok:

` $ hg version --svn`
` Mercurial Distributed SCM (version 1.6.4)`

` Copyright (C) 2005-2010 Matt Mackall `<mpm@selenic.com>` and others`
` This is free software; see the source for copying conditions. There is NO`
` warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.`

` hgsubversion: 1.1.2+101-bfb88a304ebe`
` Subversion: 1.6.12`
` bindings: Subvertpy 0.7.4`

### Cloning the nixpkgs svn repository

This \*WILL\* take a \*LONG\* time (about 10-20 minutes)

` $ mkdir -p ~/some/path/`
` $ cd ~/some/path`
` $ hg clone `[`https://svn.nixos.org/repos/nix/nixpkgs/trunk`](https://svn.nixos.org/repos/nix/nixpkgs/trunk)` nixpkgs-hg`

-   Once done you \*MUST\* ensure that all file permissions are the same as in /etc/nixos/nixpkgs (mercurial will apply commits from the beginning of the history to the latest revision to rebuild the tree, "forgetting" about file permissions in the way).
-   Failing to do so will result in your tree producing derivations with different hashes than the official nixpkgs/trunk (and by consequence the inability to use binaries built by hydra, i.e. you'll need to re-bootstrap your nixos entierely). Worst the inability to bootstrap the system, as the files in question should really have the executable flag activated (see below).
-   In my case the only files with wrong permissions were those executable needed for bootstrapping the system, very bad.
-   Make sure that at least those files have these permissions (notice the executable flag).

` $ ls -l pkgs/stdenv/linux/bootstrap/i686/`
` totale 312K`
` -rwxr-xr-x 1 imc users  73K 13 ott 21:54 bzip2`
` -rwxr-xr-x 1 imc users  14K 13 ott 21:54 cpio`
` -rwxr-xr-x 1 imc users 120K 13 ott 21:54 curl.bz2`
` -rw-r--r-- 1 imc users  289 13 ott 21:54 default.nix`
` -rwxr-xr-x 1 imc users 8,0K 13 ott 21:54 ln`
` -rwxr-xr-x 1 imc users 9,4K 13 ott 21:54 mkdir`
` -rwxr-xr-x 1 imc users  62K 13 ott 21:54 sh`

### Staying up to date

` $ hg pull`

### Diffs between svn and your version

` $ hg diff --svn -r`<hg LOCAL revision>

(will show the svn revision of modified files)

### Commit to svn (not tested!)

` $ hg push`