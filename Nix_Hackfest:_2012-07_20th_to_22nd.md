---
title: Nix Hackfest: 2012-07 20th to 22nd
permalink: /Nix_Hackfest:_2012-07_20th_to_22nd/
---

Munich, exact location to be announced.

Leave a comment here if you are interested in attending a [hackfest](http://en.wikipedia.org/wiki/Hackathon) or participating remotely. Include your name or nick and mention what you would like to work on in particular. Try to include some detail if the topic requires an introduction for others, pointers to mailing list discussions, tickets etc. could also be helpful. We don't all have to work on the same thing, but it is good to take advantage of having people around to flesh things out.

goibhniu : Improve python support
chaoflow : Improve python support
qknight : Improve python support
yournick : your topic(s) of interest

Improve python support
----------------------

Work on solving some of the following issues

-   Python package impurity (stop packages from trying to install dependencies)
-   Run the package tests (currently most of them are disabled because they don't work)
-   Write nix tests which test that the python modules work (to some degree)
-   Look at generating nix expressions from pypi (see MarcWeber's work on this)

<!-- -->

-   <https://github.com/garbas/python2nix>
-   <https://github.com/chaoflow/nixpkgs> (python branch)
-   <https://gitorious.org/nixpkgs-python-overlay>

Known issues:

-   Overzealous wrapPythonPrograms, also wraps included modules (just use wrapProgram?)

Considerations:

-   Should we use distribute as the basis for installing python packages?
