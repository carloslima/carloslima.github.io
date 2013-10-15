---
layout: post
title: "git submodules: how (not) to use it."
date: 2013-10-16 01:35
comments: true
categories: git
---

# Intro (or What's a git submodule?)

On our system, /dev/SuperFoo/sub-bar is a submodule of SuperFoo.git repository.

    carlos@carlosdev /d/SuperFoo (master)> git log -1
    commit c2abbad057a02a2bd0d7c1e1c74048da6ef88234
    Author: Chuck Norris <chuck@example.com>
    Date:   Thu Oct 10 02:35:16 2013 +0000

        Updated to latest translation

&nbsp;

    carlos@carlosdev /d/SuperFoo (master)> cd sub-bar
    carlos@carlosdev /d/S/sub-bar ((2279a91))> git log -1
    commit 2279a9187b023f79cf274f52a76fe5059f119914
    Merge: fc57e1a a3110e8
    Author: Chuck Norris <chuck@example.com>
    Date:   Thu Oct 10 02:24:15 2013 +0000

        Merge branch 'master' into currenttranslations

# so what?

They are disconnected, despite being shown/accessible as a regular subdirectory of your main project.

## again?

    carlos@carlosdev /d/SuperFoo (master)> vi sub-bar/templates/home/login.html.haml cgi/fooish.cgi
    ..(change change change)..
    carlos@carlosdev /d/SuperFoo (master)> git status

    # On branch master
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #   (commit or discard the untracked or modified content in submodules)
    #
    #       modified:   sub-bar (modified content)
    #       modified:   cgi/fooish.cgi
    #
    no changes added to commit (use "git add" and/or "git commit -a")

sub-bar/ files *are not* part of your repository!

Look again:

    carlos@carlosdev /d/SuperFoo (master)> git diff
    diff --git a/sub-bar b/sub-bar
    --- a/sub-bar
    +++ b/sub-bar
    @@ -1 +1 @@
    -Subproject commit 2279a9187b023f79cf274f52a76fe5059f119914
    +Subproject commit 2279a9187b023f79cf274f52a76fe5059f119914-dirty
    diff --git a/cgi/fooish.cgi b/cgi/fooish.cgi
    index c607f79..2b1ce17 100755
    --- a/cgi/fooish.cgi
    +++ b/cgi/fooish.cgi
    @@ -1,4 +1,3 @@
    -#!/usr/bin/perl
     
     use strict;
     use warnings;
    carlos@carlosdev /d/SuperFoo (master)> 

# Ok, got it. But what's the deal? (or, What could possibly go wrong?)

This *happened* before, more than once. Not hypothetical.

    carlos@carlosdev /d/SuperFoo (bugfixes)> git status
    # On branch bugfixes
    # Your branch is behind 'origin/bugfixes' by 1813 commits, and can be fast-forwarded.
    #
    nothing to commit (working directory clean)
    
    carlos@carlosdev /d/SuperFoo (bugfixes)> git rebase -p
    Successfully rebased and updated refs/heads/bugfixes.

All good. Let's go fix our ticket!


    carlos@carlosdev /d/SuperFoo (bugfixes)> vi lib/README.txt 
    carlos@carlosdev /d/SuperFoo (bugfixes)> git diff
    diff --git a/sub-bar b/sub-bar
    index 8b6fa45..eac1423 160000
    --- a/sub-bar
    +++ b/sub-bar
    @@ -1 +1 @@
    -Subproject commit 8b6fa45a258e37293d472c81844a3c37e921b6f9
    +Subproject commit eac1423fcb1812e0ff958712231dddc06687733c
    diff --git a/lib/README.txt b/lib/README.txt
    index 293a71f..e9a6e01 100644
    --- a/lib/README.txt
    +++ b/lib/README.txt
    @@ -1,4 +1,4 @@
     This directory holds shared code between all sub-projects. Please keep it clean.
     
    -See https://internal.example.com/wiki/code-rules
    +See https://example.thirdparty-inthecloud.net/wiki/code-rules


Looks good, simple enough.


    carlos@carlosdev /d/SuperFoo (bugfixes)> git commit -am 'Fix wrong url'
    [bugfixes 9ecfc7d] Fix wrong url
     2 files changed, 2 insertions(+), 2 deletions(-)

Cheers! We just rolled back sub-bar by 1 month.


    carlos@carlosdev /d/S/sub-bar ((eac1423))> git log -1 8b6fa45a258e37293d472c81844a3c37e921b6f9
    commit 8b6fa45a258e37293d472c81844a3c37e921b6f9
    Author: Michelle Yeoh <mi@example.com>
    Date:   Thu Oct 3 11:52:38 2013 +0000
    
        Translated using Weblate (Portuguese)
        
        Currently translated at 100.0% (448 of 448 strings)
&nbsp;

    carlos@carlosdev /d/S/sub-bar ((eac1423))> git log -1 eac1423fcb1812e0ff958712231dddc06687733c
    commit eac1423fcb1812e0ff958712231dddc06687733c
    Author: Angelina Jolie <angie@example.com>
    Date:   Mon Sep 2 02:10:57 2013 +0000
    
        Tidy

Next time bugfixes gets released, the version of sub-bar in production will be from 1 month ago.

Not just minor changes..

    carlos@carlosdev /d/S/sub-bar ((eac1423))> git diff --stat eac1423..8b6fa45
    .. snip ..
    78 files changed, 5871 insertions(+), 1435 deletions(-)

# Why did that happen?

Because it is a separate repository, git only tracks three things about the submodule:

* url of the external repository
* path where the files will be put/cloned
* the commit id from that repository that you want to use.
  
They are:

    carlos@carlosdev /d/SuperFoo (bugfixes)> cat .gitmodules
    [submodule "sub-bar"]
        path = sub-bar
        url = git@bitbucket.org:exampledotcom/sub-bar.git
    
    carlos@carlosdev /d/SuperFoo (bugfixes)> git ls-tree bugfixes
    ..snip..
    100644 blob 2d918f0b794c059d6900e7211cd34e22aa395a77    Makefile   ← file
    040000 tree 486d5dafd1d4fd7ff4f8cf3b273d5912e84666ae    api        ← directory
    160000 commit eac1423fcb1812e0ff958712231dddc06687733c  sub-bar    ← submodule (commit id to use)
    ..snip..

When we did the rebase, the changes included updates to the version of sub-bar that the SuperFoo repository should be using.  
But git *DOES NOT* checkout that new version inside the sub-bar directory.

Git knows we should be using commit 8b6fa45  
But on sub-bar directory we have eac1423 (from before the rebase)

It doesn't know (or care) if the commits are newer, older, etc.  
So, when we did `git commit -am'...'` without paying attention to that sub-bar mention that appeared on `git diff` or `git status`, we told git that we want it to use commit eac1423, which is what we have checked out at the moment.

# This is really confusing. I don't care about sub-bar and I'm not touching it, how do I avoid breaking things?

First and foremost, when `git status` or `git diff` shows sub-bar, *do not ignore it*.

The root cause is that when you move your repository to a different state, git won't automatically move the submodule to the new expected commit.  
So we fix it with `git submodule update`.

The name might be counter-intuitive, but `git submodule update` doesn't change/write/commit/push the submodule, it just runs a checkout of the expected commit.

Second, don't `git add sub-bar`;  
(Unless updating the submodule to a new version is what you're trying to do.)

## Caveats

### Bitbucket
Bitbucket (as of 2013-10-10) won't show changes to submodules on their diff.  
That means reviewers won't be able to see that you made a mistake!

### git commit -a / git commit .
Please, don't do it. Ever.  
This will happily commit every change you have on disk.  
It is an horrible practice, much more dangerous when submodules are involved.  
Always review your changes with `git status` and `git diff`.  
Don't commit anything you didn't mean to.
