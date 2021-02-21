---
layout:	post
title:	"Git merge vs Git Rebase"
description: "Easy to understand tutorial clarifying git rebase and git merge. Although they might serve the same purpose of combining multiple branches, yet they are subtle nuances to understand. "
date:	2019-11-16
categories: [ 'Git' ]
image: '0_-ExPwJS4GXXNwbyQ'
author: admin
---

A complete discussion about git rebase vs git merge (and squash merge)

{% include pictures.html img="0_-ExPwJS4GXXNwbyQ" alt="Git merge vs Git Rebase" %}
***

### Table of Contents

1. Introduction
2. Git non fast-forward merge (A Type of Explicit Merge)
3. Git fast-forward merge(A Type of Implicit Merge)
4. Git Rebase (Another Type of Implicit Merge)
5. Git squash merge (An Explicit Merge followed by rebasing or fast forward)
6. Why such concern over rebase? A practical example
7. Guideline

***

### Introduction

In Git, there are two prominent ways to converge multiple branches of development — git merge and rebase.

With all the references from the experts and articles, everyone believes “*Rebase could cause serious problems.*” so we will discuss what merge and rebase strategies are? How to perform them, which one to choose and when?

***

### Git Merge

Merging takes the contents of a source branch and integrates them with a target branch.

* Merge is always a forward-moving change record
* Merging (except squash) is **non-destructive**, commit Ids are not changed in any way.

***

### 1. Explicit git merge (a.k.a non fast forward merge)


> Explicit merge creates a **new** commit.That is a very important thing to remember and one that is elusive to the newcomers. It is a simple commit with one difference — it has **two parents**. All other regular commits have only one.

How do you check that? Fire up the terminal and a merge commit would look like below in *git log* .
```
  commit 229b6993346adae1e5b095c356b4af11dd1bb9da  
  Merge: 7222f6a c952e91  
  Author: Pranay Kumar <pranayaggarwal25@gmail.com> 
  Date: Sat Oct 19 20:24:00 2019 +0530
  Merge branch 'master' of github.com:pranayaggarwal/enumifier
```

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*DHmWhgEMZOSW3OVz.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Sample animation for Explicit git merge*

#### Pros

* Preserves complete history and chronological order.

#### Cons

* Commit history can become polluted by lots of merge commits.
* Debugging using git bisect can become harder.

***

### 2. Implicit git merge via fast forward merge


> It can only happen if there are no more commit made in source i.e. HEAD is not diverged.

That’s why the implicit merge can be completed without an explicit merge commit: it literally just fast-forwards the branch label to the new commit as shown below.

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*GhW5WSCRH1dneU6c.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Sample animation for Implicit git merge*

#### Pros

* No extra commit is made so commit history doesn’t get polluted.
* Converges possibly capable of being merged branches in a cleaner way.

#### Cons

* Fast forward merge will still lose some context of those commits as part of an earlier feature branch.

#### How to do git merge

In the event that you require a merge commit during a fast forward merge for record-keeping purposes, you can execute git merge with the --no-ffoption.

{% include pictures.html img="0_6OWQ6E6bT-TmuEvJ" alt="git merge — no-ff vs git merge" %}*git merge — no-ff vs git merge*

Merge the master branch into the feature branch using the checkout and merge commands.
```shell

$ git checkout feature  
$ git merge master

(or)

$ git merge master feature

```
This will create a new “**Merge commit**” in the feature branch that holds the history of both branches.

***

### Git Rebase (Implicit merge via rebase way)

Rebase is **recreating your work from one branch onto another**.

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*JRt9VF_osaAoVwKg.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>

Read this again, slowly: **new commit for every old one, with the same changes**.

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*1VkO0oTn-PPPzJmx.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Sample animation for git rebase*

Used this way, one can indeed apply some commits to master without creating a merge commit. This procedure completely loses the context of where those commits come from, unfortunately.

#### Pros

* Streamlines a potentially complex history
* Manipulating a single commit is easy (e.g. reverting)
* Avoids merge commit “noise” with busy branches.

#### Cons

* Squashing the feature down to a handful of commits can hide the context
* Rebasing public repositories can be dangerous when working as a team.
* It’s more work, Rebasing with remote branches requires you to *force push*. The biggest problem people face is they force push but haven’t set git push default. This results in updates to all branches having the same name, both locally and remotely, and that is **dreadful** to deal with.

#### Interactive Rebasing

This allows altering the commits as they are moved to the new branch. Typically this is used to clean up a messy history before merging a feature branch into master

#### How to do it

Rebase the feature branch onto the master branch using the following commands.

```
  $ git checkout feature  
  $ git rebase master // use with -i option for interactive
```

***

### Git squash merge

#### (usually followed by implicit merge)

A third way to move changes is to *squash* all feature branch’s commits into a single commit before performing an implicit merge *fast-forward* merge or *rebase*.

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*0BgII1PmO8JzjnXD.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Sample animation for squash merge*

If you use explicit merges this need does not arise because the explicit merge commit allows you to reconstruct what was in the feature branch and its entire evolution.

#### Pros

* Keeps the mainline branch history linear and clean
* It isolates the entire feature in a single commit.
* If used with a fast-forward merge, it can give you an advantage of both explicit merge and rebase.

#### Cons

* It loses insight and details on how the feature branch developed throughout.
* You might be compelled to keep the original, *unsquashed*, feature branch around for historical reasons.

#### How to do it

```
  $ git merge --squash feature  
  // After this you would need to explicitly commit this squashed commit
```

### Why such concern over rebase?

> **Merge preserves history whereas rebase rewrites it.** 

Consider the case where a dependency that is still in use on feature has been removed on *master*. When *feature* is being rebased onto *master*, the first re-applied commit will break your build, but as long as there are no merge conflicts, the *rebase* process will continue uninterrupted. The error from the first commit will remain present in *all* subsequent commits, resulting in a chain of broken commits.

This error is only discovered after the *rebase* process is finished, and is usually fixed by applying a new bugfix commit *g* on top.

![](https://cdn-images-1.medium.com/max/800/0*doyGQuGtuXWa3jMF.gif)
*Example of failed rebasing*

Even If you do get conflicts during rebasing however, solving conflicts in the middle of rebasing a long chain of commits is often confusing, hard to get right, and another source of potential errors.

This way, new errors are introduced when you rewrite history, and they may disguise genuine bugs that were introduced when history was first written. In particular, this will make it harder to use **Git bisect** which performs a bisection search through the history, identifying the commit that introduced the bug —.

```
  git bisect run <yourtest.sh>
```

<div class="vidWrapper">
  <video style="max-width:100%" autoplay muted loop>
    <source src="/img/0*cfutfYn25YFEgBvM.mp4" type="video/mp4">
    Your browser does not support the video tag.
  </video>
</div>
*Example of a failed Git bisect*

if we’ve introduced additional broken commits during rebasing (here, *d* and *e*), bisect will run into trouble. In this case, we hope that Git identifies the commit *f* as the bad one, but it erroneously identifies dinstead, since it contains some other error that breaks the test.

***

### Guideline

Use **rebase** when:

1. You have a need to merge local changes and don’t need an exact history. Why litter it with merge commits?
2. You prefer a linear history and use [git bisect](https://five.agency/cut-your-way-through-problems-with-git-bisect/) very often (it can get confused with a non-linear history).


Use **merge** when:

1. You have shared some of the changes with others and it’s important not to break their repositories. git rebase changes a lot of history so a normal merge is much safer and cleaner for others.
2. You care about history and development tracks.
Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comments to improve this article.  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com), Have a great day!

***

### References

1. [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model)
2. [Git merge debate](https://blog.developer.atlassian.com/pull-request-merge-strategies-the-great-debate/)
3. [Why you should stop using Git rebase](https://medium.com/@fredrikmorken/why-you-should-stop-using-git-rebase-5552bee4fed1)