---
title: Samba
permalink: /Samba/
---

this guide will help you on how to use samba on nixos. as assumed it's quite trivial ;-)

/etc/nixos/configuration.nix
----------------------------

` ...`
` services.samba.enable = true;`
` services.samba.securityType = "share";`
` services.samba.extraConfig = ''`
`   workgroup = WORKGROUP`
`   server string = albstadt`
`   netbios name = albstadt`
`   security = share `
`   #use sendfile = yes`
`   #max protocol = smb2`
` `
`   [rw-files]`
`     comment = Temporary rw files `
`     path = /storage`
`     read only = no`
`     writable = yes`
`     public = yes`
` '';`
` ...`

afterwards apply the changes made to configuration.nix
------------------------------------------------------

` nixos-rebuild switch`

samba should startup afterwards

links
-----

-   <http://hydra.nixos.org/build/1457802/download/1/nixos/manual.html>
