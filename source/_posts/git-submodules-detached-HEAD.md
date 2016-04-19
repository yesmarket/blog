---
title: git submodules detached HEAD
language: default
date: 2016-04-20 00:51:08
tags:
	- git
	- GitHub-pages
---

For my hexo blog, I've got 2 git repositories; one for the static site generator, which hosts the actual blogs, the bootstrap theme etc., and a second one for the generated content. The latter is set up as a [GitHub-pages](https://pages.github.com) site hosted directly from my repo - all I need to do is <code lang="sh" linenumbers="off">git push</code> and my changes are live - it's awesome, but I digress.

Up until recently, I've been maintaining 2 completely seperate git repositories. I got the idea of making the generated content a submodule of the static site generator after reading about submodules in [ProGit](http://labs.kernelconcepts.de/downloads/books/Pro%20Git%20-%20Scott%20Chacon.pdf). The book gives a few examples of where submodules might come in handy such as when you're creating an application using a framwork that you're customising at the same time you're building the application.

An important thing to note when dealing with submodules is that basically they're just a pointer to a particular commit of the submodule's repository; they don't (by default) deal with branches. This caused me quite a few headaches, no pun intended, whereby the submodule was always in a detached HEAD state after running <code lang="sh" linenumbers="off">git submodule update --remote</code>. I tried to solve this issue by manually editing the .gitmodules file and setting up my submodule to track the master branch:

    [submodule "public"]
	    path = public
	    url = git@github.com:yesmarket/yesmarket.github.io.git
	    branch = master

However, this didn't work, that is, without a little help. I found a git command to recursively loop through each of the submodules in the repo, checking out the branch specified in the .gitmodules file. To make things as simple as possible I 'aliasised' some commands in my global git config;

    submodule-trackbranch = "!git submodule foreach -q --recursive 'branch=\"$(git config -f $toplevel/.gitmodules submodule.$name.branch)\"; git checkout $branch'"
    upsub = !git submodule update --init --recursive --remote && git submodule-trackbranch

Now when I clone a repo with a submodule, I can then run <code lang="sh" linenumbers="off">git upsub</code> to update the submodule(s) to match what the superproject expects and then checkout the correct branch as specified in the .gitmodules file. This means I don't end up with a detached HEAD.

<div style="float:left;">
	![](/images/detached-head.jpg)
</div>
