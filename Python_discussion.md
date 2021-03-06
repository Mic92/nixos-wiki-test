---
title: Python discussion
permalink: /Python_discussion/
---

python basics
-------------

-   **\*.pth** files


Python supports \*.pth files which contain the paths to the parent directories of your python modules one per line. Those files should be placed in site-packages and can be called whatever you want them to call.

-   PYTHONPATH


The other way is to set additional paths for python just before running the externally kept modules. This is done by setting the python paths to the environment variable PYTHONPATH. Note again that python paths point not to the modules themselves, but to their parent directories!

-   PYTHONUSERBASE


?

in general see \[6\]

general python integration into nixos
-------------------------------------

### requirements

-   mix python 2.x and 3.x in the same profile


marcweber and qknight **vote against that**. it makes packaging complicated: to mix python2 and python3? We could introduce NIX_PYTHON_SITES_2 and NIX_PYTHON_SITES_3 or such.

-   pyLibA (python wrapper for a library) is used by pyLibB: nix expressions must be able to use either

### proposed nixos environment variables

-   NIX_PYTHON_SITES

qknight's problems
------------------

### packaging virt-manager

see **problem 2** in \[5\]

### python-gudev

i have problems packaging python-gudev:

` gudevmodule.c:19:23: fatal error: pygobject.h: No such file or directory`

but looking at the exports (nix-env -i python-gudev -K; source env-vars) i can see:

`-I/nix/store/lqsihhkhl7rhr18x7qiviihyp2byc5gw-pygobject-2.27.0/include`

but in that folder there is only pygtk-2.0 folder:

` ls -la /nix/store/lqsihhkhl7rhr18x7qiviihyp2byc5gw-pygobject-2.27.0/include`
` pygtk-2.0/`

so in that folder i find:

` ls -la /nix/store/lqsihhkhl7rhr18x7qiviihyp2byc5gw-pygobject-2.27.0/include/pygtk-2.0/`
` pyglib.h  pygobject.h`

### the problem

as discussed in \[1\] and quoting \[2\]

`   # All python code is installed into a "gtk-2.0" sub-directory. That`
`   # sub-directory may be useful on systems which share several library`
`   # versions in the same prefix, i.e. /usr/local, but on Nix that directory`
`   # is useless. Furthermore, its existence makes it very hard to guess a`
`   # proper $PYTHONPATH that allows "import gtk" to succeed.`

show how should i package python-gudev then? the source for python-gudev (which produces the above error) can be found at \[4\]

links
-----

-   \[1\] <http://old.nabble.com/Python-3-to22076291.html#a22119842>
-   \[2\] <https://nixos.org/repos/nix/nixpkgs/trunk/pkgs/development/python-modules/pygobject/default.nix>
-   \[3\] <http://dpaste.com/756606/>
-   \[4\] <https://github.com/NixOS/nixpkgs/blob/master/pkgs/development/python-modules/python-gudev/default.nix>
-   \[5\] <http://permalink.gmane.org/gmane.linux.distributions.nixos/8732>
-   \[6\] <http://djangotricks.blogspot.de/2008/09/note-on-python-paths.html>
