---
title: For The Uninitiated
permalink: /For_The_Uninitiated/
---

This page is ment to be a long list command line applications and functions that some people may not know how to use. For instance how to pipe the output of a command to another command (ex. **lspci |grep Network**). It is ment to be both a learning tool for bash, nix-expressions and NixOS.


**Under construction**. As such this page does not yet need a structure but is merely present so that it can be shared and edited by the community.

<!-- -->


**This page is a part of the [UDX Initiative](/UDX_Initiative "wikilink")** which focuses on easily understood, simple and clear explanations and documentation to ensure the least amount of communication faults between User & Developer

### The Basics

A command line is a line of text which the user can write text in to run applications and manipulate the environment and operating system, which then usually outputs one, several or many lines of text to illustrate a task or information in a systematic and structured way.

All commands in the command line have options to them signified by either one line for short (**-v**) or two lines for the name of the function (**--verbose**). When using verbose we are asking the command to not filter or omit any information, but to give as much information as possible (or to be *verbose*).

### Filtering Commands

*Cut* [1](http://man.he.net/?topic=cut&section=all)
Cut allows you to cut out certain bytes, characters and fields provided by output.

:; <small>*To cut away the rest of the characters in a line after **32** characters*</small>



`ls /nix/store/ |cut -c -32`

:; <small>*To cut away the first **34** characters of a line:*</small>



`ls /nix/store/ |cut -c 34-`

