---
title: OpenGrok
permalink: /OpenGrok/
---

OpenGrok is source code search and cross reference engine that can be used both from command line and from the web. NixOS actually provides the package but not the service.

This guide explains how to set up the OpenGrok web interface through tomcat. We also assume for simplicity that the the opengrok data is owned by the tomcat user, and that we will be indexing git repositories.

First of all enable tomcat, and let the opengrok webapp find git:

services.tomcat.enable = true; jobs.tomcat.path = \[ pkgs.gitFull \];

Now install opengrok and deploy it. Opengrok will automatically detect tomcat and deploy the .war under /source:

1.  nix-env -i opengrok
2.  OpenGrok deploy

You should now be able to open <http://localhost:8080/source> .

Now let's create the necessary opengrok directories and fetch some source to be indexed:

1.  mkdir /var/opengrok/src
2.  cd /var/opengrok/src
3.  git clone <https://github.com/NixOS/nix.git>
4.  chown -R tomcat /var/opengrok

Each directory under /var/opengrok/src is a project to be indexed.

Now run the indexer:

1.  OpenGrok index

Note that until it hasn't finished, you will not be able to search anything in the web interface.