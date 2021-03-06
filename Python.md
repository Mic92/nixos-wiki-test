---
title: Python
permalink: /Python/
---

Installing Python interpreters
==============================

` nix-env -i python (installs default python, currently 2.7)`
` nix-env -i python33`
` nix-env -i python32`
` nix-env -i python26`
` nix-env -i pypy`

Installing Packages
===================

` nix-env -i pythonPackages.pyramid (installs pyramid with default python, currently 2.7)`
` nix-env -i python33Packages.pyramid (installs pyramid with python 3.3)`

Rationale of not polluting global site packages
===============================================

Nix does not pollute the \*site-packages\* directory when you install new python software. It only provides the binary executables to run.

If you want to make a collection of python packages available to the python interpreter, you can either use \*virtualenv\* to install the packages inside

or create another nix profile, see the next section.

Packaging python software
=========================

tests
-----

It is required to run tests, as it gives instant feedback if whole environment works:

`  doCheck = true;`

In case tests fail in any way, you can leave them our (or patch them), but leave a comment note why:

`  # no tests `
`  doCheck = false;`

or

`  # tests failing, see `[`http://`](http://)`...`
`  doCheck = false;`

with setup.py
-------------

Nix provides function \*\*buildPythonPackage\*\* with tons of examples in \*\*nixpkgs/pkgs/top-level/python-packages.nix\*\*.

package providing python modules
--------------------------------

If you want to provide python modules for reverse dependencies to pick it up, add postInstall phase:

`  cp -v src/swig/python/mlt.py $out/lib/${python.libPrefix}/site-packages/`

Creating environments with packages
===================================

For a general overview of creating development environments, see [Howto_develop_software_on_nixos](/Howto_develop_software_on_nixos "wikilink").

Start by defining your environment:

`$ cat configuration.nix`
`{`
`  packageOverrides = pkgs: with pkgs; {`
`    myPythonEnv = pkgs.myEnvFun {`
`        name = "mypython";`
`        buildInputs = [`
`          python33`
`          python33Packages.pyramid`
`          python33Packages.jinja2`
`        ];`
`    };`
`  };`
`}`

Install it:

` $ nix-env -i env-mypython`

Load the environment and start hacking:

` # load-env-mypython`