---
title: Vimprobable
permalink: /Vimprobable/
---

Vimprobable
===========

Vimprobable is a web browser that behaves like the [Vimperator plugin](http://www.vimperator.org/) available for Mozilla Firefox. It is based on the WebKit engine (using GTK bindings). The goal of Vimprobable is to build a completely keyboard-driven, efficient and pleasurable browsing-experience. Its featureset might be considered "minimalistic", but not as minimalistic as being completely featureless.

Vimprobable is available in two versions, that is vimprobable and vimprobable2. As of now only vimprobable2 is available and vimprobable will be used synonymously with vimprobable2 from now on.

Installation
------------

`$ nix-env -i vimprobable2`

Usage
-----

vimprobable is able (and intended to be) completely keyboard-driven. Citing from vimprobable's manual page here are a few basic commands that should keep you going.

| Key | Action                                               |
|-----|------------------------------------------------------|
| o   | Insert URL from keyboard and load it                 |
| t   | Insert URL from keyboard and load it into new window |
| j   | Scroll down                                          |
| k   | Scroll up                                            |
| d   | Close current window                                 |
||

Be sure to read `man vimprobable2` eventually to learn all the other important commands.

Configuration
-------------

vimprobable looks for configuration files in `$HOME/.config/vimprobable`. The basic configuration is done in `vimprobablerc`. To create this file issue the following commands.

`$ mkdir -p $HOME/.config/vimprobable`
`$ touch $HOME/.config/vimprobable/vimprobablerc`

See `man vimprobablerc` for configuration options.

### Search engines

vimprobable can be configured to use a variety of search engines. A benefit from configuring a search engine is that you can use them for URLs. For example you can just type `g NixOS` to search for *NixOS* on Google. The following example sets up the search engines Google (shortcut `g`), Wikipedia (shortcut `w`) and NixOS wiki (shortcut `wn`).

`$ touch $HOME/.config/vimprobable/searchengines`
`$ cat << EOF`
`g  `[`https://google.com/search?q=%s`](https://google.com/search?q=%s)
`nw `[`https://nixos.org/wiki/Special:Search?search=%s`](https://nixos.org/wiki/Special:Search?search=%s)
`w  `[`https://secure.wikimedia.org/wikipedia/en/w/index.php?search=%s`](https://secure.wikimedia.org/wikipedia/en/w/index.php?search=%s)
`$ EOF > $HOME/.config/vimprobable/searchengines`

See `man vimprobable2` for default search engines that come with vimprobable.

### Tab support

vimprobable doesn't offer tab support. However, you can use [tabbed](http://tools.suckless.org/tabbed) for that. Install tabbed and create an alias to vimprobable2 as following.

`$ nix-env -i tabbed`

`$ alias vimprobable2="vimprobable2 -e \"\$(tabbed -d)\""`

You can switch tabs by pressing `gt` or `gT`.

References
----------

-   [Official website](http://sourceforge.net/apps/trac/vimprobable/)
