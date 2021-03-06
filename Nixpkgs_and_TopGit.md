---
title: Nixpkgs and TopGit
permalink: /Nixpkgs_and_TopGit/
---

What is TopGit about ?
----------------------

TopGit is used to manage a set of patches on top of upstream branches. Those patches are usually referred to as "topics" because each branch is created to implement a small story. Such a topic-branch will end up as being one commit later. There are alternative solutions such as quilt, stgit. Note: Mercurial and others have similar extensions AFAIK. In contrast to keeping piles of patches TopGit is collaborative. This means you and others can work on a topic on different machines easily until you feel the patch is ready - maybe this never happens. Because its collaborative TopGit was designed to follow branches upstream only. WARNING: Going back to a previous state is possible but very hard. So if you don't know what will happen at the beginning you may want to backup your repository. You only mess up the TopGit branches which can be deleted and recreated though.

So how does TopGit work?

TopGit adds two files: .topmsg and .topdeps The first file contains a description of the topic. The second tells TopGit about how to follow upstream. (On which branches does this topc depend upon) .topmsg will be end up being the commit message when exporting the topic and feeding it upstream. Its good practise starting all TopGit branches with t/ or tg/.

step by step example (without TopGit)
-------------------------------------

Usually you follow upstream. Eg `
  # create a topic branch
  git checkout -b MY_BRANCH
  # hack and commit two times
  git commit -am 'first change H1'
  git commit -am 'next change H2'

  # follow upstream by updating master, then merging master into MY_BRANCH
  git checkout master; git pull;
  git checkout MY_BRANCH; git merge master
`

This last step merging with master is very important. It ensures that your patche(s) doesn't drift away too much. However it pollutes history a little bit.

So development usually looks like this after some time: (\* denotes a merging trunk into MY_BRANCH). ``
  X -  X - A   - B   -   C   -    D -  [master]
        ` H1 - H2 `*  - H3 ` * -  [MY_BRANCH]
 ``

step by step example (with TopGit)
----------------------------------

Simple speaking: TopGit automates this task for you:

perquisites: install git and gitAndTools.topGit

`
  TopGit create tg/MY_BRANCH
  # give a description of this topic branch.
  $EDITOR .topmsg
  git add .topmsg
  git commit

  # work on the topic branch:
  # HACK ..
  git commit -am 'initial not very well tested version ..'

  # update with upstream:
  tg update

  # change message
  $EDITOR .topmsg
  git commit -am 'changing topic message'

  # export topic(s) to branch
  tg export [--linearize] test-export

  # export topic(s) to patch seriees:
  tg export --quilt DIRECTORY
`

trees of topics
---------------

Of course if you would only track a simple topic you would not start using TopGit. topics can depend on others:

`
  tg create tg/adding-new-bulider; # edit .topmsg and commit
  tg create tg/adding-package-depending-on-new-builder# edit .topmsg and commit.
  git checkout tg/adding-new-bulider;
  # HACK X
  git commit -m 'message is not that important. .topmsg will be used anyway..'
  git checkout tg/adding-package-depending-on-new-builder
  tg update # merge HACK X updates into this topic-branch
`

Now our topic tree looks like this: ``
  msater
    `- tg/adding-new-bulider
     ` - tg/adding-package-depending-on-new-builder
 ``

You'd use tgs summary --graphviz feature to create this graph: `
  dotty <( tg summary --graphviz)
`

Using tg depend add BRANHCH_NAME you can make a topic depend on multiple upstream branches or other topics.

TopGit internals
----------------

Each topic branch existst of two branches: a merge base tracking dependencies and the topic branch. When running tg update TopGit first merges the base with all dependencies. Then it merges the topic-branch into this base. Because each topic branch has a base and because each topic may have multiple dependencies its hard to undo an operation. So you understand my warning now :)

The collaborative part
----------------------

`
  # get all merge bases and topic branches:
  tg remote origin --populate

  # push a topic (and its dependencies) to a remote git repository:
  tg push -r origin t/branch-name
`

Workflow: `
  tg remote origin --populate # update remote branches
  tg update # update dependencies, then merge base with deps, then merge topic branch and its base.
  # HACK
  git commit -m 'more work has been done'
  tg push -r origin t/this-topic-branch
`

Of course when running tg update you run the risk to get merge trouble even if there are no conflicts. I think git folks just live with it. They fix the failure and do a new commit. Commits will be squashed when exporting TopGit branches anyway.

You should know
---------------

If different users work on a topic the information will be lost who did what because one branch will be collapsed. Best we can do is adding to the .topmsg file `
  message
  work done by name, name, name
` You never know what others will add after your work. So signing off a commit too early can be risky. (I hope that communication is fine that contributers talk before feeding such a topic upstream - So this is not a problem)

scripts
-------

I use these bash functions to speed exporting a topic branch up and creating new ones: `
  # create new topic branch and open editor, commit .topmsg file
  tgCreate () {
          tg create "$@" || {
                  echo 'failure, starting subshell'
                  $SHELL
          }
          $EDITOR .topmsg
          git add .topmsg
          git commit -m "new tg branch $1"
  }

  # export checked out topic branch to export/BRANCH_NAME assuming the topic was
  # prefixed by t/
  tgExport () {
          local branch=$(cat .git/HEAD |  sed 's@ref: refs/heads/\(t/\)\?\(.*\)@\2@')
          local prefix=export/
          local exported=${prefix}$branch
          git branch -D $exported
          tg export "$@" $exported
  }

`

Questions
---------

contact MarcWeber on irc and ask him to improve this article!