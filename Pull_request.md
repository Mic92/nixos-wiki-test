---
title: Pull request
permalink: /Pull_request/
---

What is a github pull request ?
-------------------------------

Its a way to tell a github project that you've created some changes. Maintainers can easily review, comment on and merge your changes.

How to create a git pull request
--------------------------------

1) Create a github account. Create a new remote location:

`   # add your github clone as remote location:`
`   YOUR_GITHUB_NAME=fill-in`
`   git remote add $YOUR_GITHUB_NAME "git@github.com:$YOUR_GITHUB_NAME/nixpkgs.git"`

2) git commit your change

`   git add FILE_LIST # eventually use git add --patch`
`   git commit`

`   # verify everything is ok - no unexpected dangling files git does not know about yet:`
`   git status`

3) submitting your change, push to your repository:

`   # recommended: create a topic branch, so that this change`
`   # can be submitted independently from other patches:`
`   git checkout -tb submit/your-topic-name`
`   git push $YOUR_GITHUB_NAME submit/your-topic-name`

goto gituhb.com/your_name -&gt; create pull request

Why create your own branch? You can follow upstream (master) by running "git merge master". You can "git commit --amend" fixes and "git push -f" your branch. You can always switch back to master by "git checkout master".

For long living topic branches you may want to read [Nixpkgs_and_TopGit](/Nixpkgs_and_TopGit "wikilink") which allows to squash many changes so that patches will always be easy to review.