---
title: Bower2nix
permalink: /Bower2nix/
---

Fetching bower dependencies from nix
------------------------------------

bower2nix[1](https://bitbucket.org/shlevy/bower2nix) is a simple tool for generating nix expressions to fetch dependencies specified in a bower[2](http://bower.io/) json file. A simple usage example:

    $ nix-env -iA bower2nix -f '<nixpkgs>'
    $ cat bower.json
    {
      "name": "bower2nix-test",
      "dependencies": {
        "jquery": "*"
      }
    }
    $ bower2nix bower.json bower-packages-generated.nix
    $ cat bower-packages-generated.nix
    { fetchbower }: [
      (fetchbower "jquery" "2.1.0" "*" "0484cy1mhxylzy155hwgngpajxhqab8r3wksk33id230ivggl98f")
    ]

The fetchbower function is defined in nixpkgs and is a fixed-output derivation, so importing the generated nix expression while passing fetchbower will give you a list of fixed-output derivations corresponding to your package dependencies.