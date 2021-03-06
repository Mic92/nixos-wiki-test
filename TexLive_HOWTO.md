---
title: TexLive HOWTO
permalink: /TexLive_HOWTO/
---

TexLive HOWTO
-------------

### TexLive components

There is a "core" texlive component and some additional components:

-   nixpkgs_sys.texLiveContext
-   nixpkgs_sys.texLiveBeamer
-   nixpkgs_sys.texLiveCMSuper
-   nixpkgs_sys.texLive
-   nixpkgs_sys.texLiveExtra
-   nixpkgs_sys.texLiveLatexXColor
-   nixpkgs_sys.texLivePGF

` nicpkgs_sys.texLive`

is the "core" component, other components add fonts/styles and extra functionalities.

### The problem

-   Extra tex components install extra functionalities in locations that are unknown to the core component (as the core component obviously does not depend on extra components).
-   After installing an extra component, a texlive user will notice that the extra functionalities are not found by texlive binaries (i.e. latex)
-   There is a file called "ls-R" that should contain all the tex-related files, and this file should be updated after an extra component is installed.
-   Another way is to add the .../texmf-dist// directories of extra components into the TEXINPUTS environment variable, i.e. on my system:

` export TEXINPUTS="./:/nix/store/d6xw1wangjq75dw7grh5d9318687hfhi-texlive-extra-2009/texmf-dist//:"`

would just solve the issue without much hassle. However, each time you upgrade/add an optional texlive component, you should update the TEXINPUTS environment variable, and this is bad.

### The Solution

Instead of explicitly install texLive components via /etc/nixos/configuration.nix (environment.systemPackages) or nix-env -i, one should write a nix-expression using the following function:

` texLiveAggregationFun`

cfr. [<http://www.mail-archive.com/nix-dev@cs.uu.nl/msg00996.html>](http://www.mail-archive.com/nix-dev@cs.uu.nl/msg00996.html) and [<http://www.mail-archive.com/nix-dev@cs.uu.nl/msg00998.html>](http://www.mail-archive.com/nix-dev@cs.uu.nl/msg00998.html)

-   nixos-rebuild way

edit /etc/nixos/configuration.nix and add (or edit, if environment.systemPackages is already set):

` environment = {`
`   systemPackages = [`
`     ...`
`     (let myTexLive = `
`        pkgs.texLiveAggregationFun {`
`           paths = [ pkgs.texLive pkgs.texLiveExtra pkgs.texLiveBeamer ];`
`        };`
`      in myTexLive)`
`      ...`
`   ];`
`   ...`
` };`

the "let myTexLive =" will introduce a new nix-expression using texLiveAggregationFun { paths = \[ all_your_texLive_components_here \]; }. the (let myTexLive = ... in myTexLive) will actually evaluate the function upon a nixos-rebuild switch command. After a nixos-rebuild switch, all the specified texLive components will be built and their texmf-dist// scanned, resulting in an environment with a full working texlive + optional components setup.

-   nix-env way

TODO