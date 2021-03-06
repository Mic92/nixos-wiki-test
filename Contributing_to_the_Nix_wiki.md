---
title: Contributing to the Nix wiki
permalink: /Contributing_to_the_Nix_wiki/
---

O(1)
----

On the haskell mailinglist there was some discussion about their [wiki](http://haskell.org). If you visit it it looks very crowded. However there is a good reason for it. You can access all content by O(1) access no matter whether you're an advanced user or a new user. I'd vote for O(1) access on the main wiki page as well - Marc Weber

WISH LIST
---------

What is missing on the wiki? even if you don't have time let others know who may have time what you think is missing. Should we group this by author? Should we add kind of rating?

This is a list containing random notes right now. Add items yourself here.

-   haskell on Nix howto. (Marc Weber is going to write this?)

### vim and .nix files. (Marc Weber is going to write this?)

#### use what is coming with nix (nix os)

`# add .nix filetype detection and minimal syntax highlighting support`
`ftNixSupport     = getConfig [ "vim" "ftNix" ] true;`

#### use the lisp syntax or what kosmikus provided

`mkdir -p ~/.vim/ftdetect/`
`NEW NEW NEW`
[`http://people.cs.uu.nl/andres/nix.vim`](http://people.cs.uu.nl/andres/nix.vim)
`NEW NEW NEW`
`find / -name syntax/lisp.vim`

Then use the found file an copy it to nix.vim

`cp /nix/store/jz5hjc8a8cl5x9pm40ib42bk13b4vazz-vim-7.3/share/vim/vim73/syntax/lisp.vim `

~/.vim/ftdetect/nix.vim

`echo "syntax on" >> ~/.vimrc`

**Note:** this is not exactly NIX syntax but looks much better than black/white anyway!

Test your new setup:

`vi /etc/nixos/configuration.nix # should be highlighted`

--[Invalidmagic](/User:Invalidmagic "wikilink") 10:31, 21 May 2011 (UTC)

-   contributor guide and workflows?

<!-- -->

-   python on Nix (Marc Weber is going to contribute writing down how it is and which problems are there)

<!-- -->

-   ruby on Nix (Marc Weber is going to write about what he has done)

<!-- -->

-   buildfarm stuff

` how to make packages being build`
` how to abort a build of packages (throw vs assert)`

-   trouble building a package? things which you can try

` Link list to other distros`
` where to get patches / source from.`
` special hints about special make systems such as cmake whatsoever`

-   Howto X?

` Which package managers are supported and which features of them ? eg kde, gnome etc`

-   Nix and encryption

<!-- -->

-   Howto add completion to bash and zsh?

<!-- -->

-   Howto work with nix - common practises

`   this should answer:`
`   how to find the package I'm looking for (Marc Weber is going to contribute)`

-   howto use git to track nix ?

`   I'm a fan of git :)`
`   `

-   ...

<!-- -->

-   list of advanced buliders

#### Visual Editor

Mediawiki have a nice editor that can be enabled (WYSIWYG), looks like this <https://upload.wikimedia.org/wikipedia/commons/5/58/Visual_Editor_%28linking%29.png>

It provides completion when linking to other pages and have other features that makes editing lot easier. You can try it if you have an account at wikipedia.

random notes
------------

What policies do we have ? Eg "Nix" and "NixOS" is the spelling which should be used everywhere. What about links? Should they all start with an upper case letter? Anyway. Keep this short because everyone is supposed to read it!

Do we want subsections for different projects?

Eg [NixOS:Modules](/NixOS:Modules "wikilink") using a colon (":") to separate the project and the page contents? Or does Mediawiki allow [NixOS/Modules, Don't create this page](/NixOS/Modules "wikilink") as well using a slash?

It is possible configure MediaWiki to treat slashes as a hierarchy, but I suspect we're better off using categories: \[\[/Category:NixOS|Category:NixOS\]\] It is also possible to have hierarchical categories but it gets a bit messy. I really like the look of the Semantic MediaWiki extensions <http://semantic-mediawiki.org> which allows for very rich categorization and queries: e.g. "[Concepts](http://semantic-mediawiki.org/wiki/Concepts)" (goibhniu)