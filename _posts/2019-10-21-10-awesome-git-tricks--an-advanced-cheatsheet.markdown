---
layout:	post
title:	"10 Awesome Git Tricks: An advanced cheatsheet"
date:	2019-10-21
categories: [ 'Git' ]
image: 'img/0*wASU3bLLfLP3ve7L'
author: admin
---

  Commit, Branching, Refs, Checkouts, push and pulls

![](/img/0*wASU3bLLfLP3ve7L)*Photo by Yancy Min on Unsplash*

***
### Table of Contents

1) Refs  
2) Better Commits  
3) Easier Checkouts  
4) A nicer force push  
5) Search in Git logs  
6) Submodule handling  
7) Empty commit (to trigger something maybe)  
8) Better Patches  
9) Better branch handling  
10) Create local origin branch from remote

***

### 1) Refs
```
  HEAD^    # 1 commit before head  
  HEAD^^   # 2 commits before head  
  HEAD^^^^ # 4 commits before head  
  HEAD~5   # 5 commits before head
```
Sample usage: *git show HEAD~2*   
This will show specified commit’s summary and diff.

***

### 2) Better Commits

To make sure each commit consists of only a single logical change, to divide things up so that each commit contains only the appropriate changes?   
**git add --patch** to the rescue!!
```
  git add -p
  # output will be.....
  # .....<file diff>
  Stage this hunk [y,n,q,a,d,e,?]?
```
This flag will cause the **git add** command to look at all the changes in your working copy and, for each one, ask if you'd like to stage it to be committed, skip over it, or defer the decision (as well as a few other more powerful options you can see by selecting after running the command).

***

### 3) Easier Checkouts

Similar to *git add -p*, the *git checkout* command will take a *--patch* or *-p* option, which will cause it to present each "hunk" of change in your local working copy and allow you to discard it.

This is fantastic when, for example, you’ve introduced a bunch of debug logging statements while chasing down a bug. After the bug is fixed, you can use it!
```
  git checkout -p # output will be  
  .....  
  .....<file diff>

  Discard this hunk from worktree [y,n,q,a,d,e,?]?
```
***

### 4) A nicer force-push

No matter how hard as you try to avoid it, sometimes *git push --force* to overwrite the history on a remote copy of your repository becomes necessary. You may have gotten some feedback that caused you to do an interactive rebase, or you may simply have messed up and want to hide the evidence.

One of the hazards with force pushes happens when somebody else has made changes on top of the same branch in the remote copy of the repository. When you force-push your rewritten history, those commits will be lost.

![](/img/0_Lqu7mVfSeMU_GGFx.webp)
This is where the following option comes in, it will not allow you to force-push if the remote branch has been updated, which will ensure you don't throw away someone else's work. Ain’t it nice?

*git push --force-with-lease*

***

### 5) Search in Git logs
```
  git log --grep="fixes things" # search in commit messages  
  git log -S"window.alert" # search in code
```
***

### 6) Submodules
```
  # Import .gitmodules  
  git submodule init# Clone missing submodules, and checkout commits  
  git submodule update --init --recursive# Update remote URLs in .gitmodules  

  # (Use when you changed remotes in submodules)  
  git submodule update --remote # To override local submodule change  
  git submodule update --remote --force
```
***

### 7) Empty Commit (to trigger something maybe)
```
  git commit --allow-empty -m “Trigger notification”
```
***
### 8) Better Patches

Try  
git [format-patch](https://www.kernel.org/pub/software/scm/git/docs/git-format-patch.html) -1 <sha>

OR

*git format-patch -1 HEAD*

The -1 flag tells git how many commits should be included in the patch;

*git format-patch -1 HEAD*

Apply the patch with the command:
```shell
  git am < file.patch  
  # OR  
  git apply xyz.patch
```
***

### 9) Better branch handling

Create the branch and checkout:
```
  git checkout -b <branch_name>   
  # -b for creating the branch if not exists
```
Delete branch locally and on remote

```
  #Local delete  
  git branch -D <branch_name> # -D for forceful delete, -d for soft


  #Push deletion on remote and thus deleting the remote branch  
  git push <remote_name> --delete <branch_name>
```

![](/img/0_jEADYPXfqHA2p14B.webp)*Delete remote branch*

***

### 10) Create local origin branch from remote
```
  git fetch origin  
  git checkout -b <remote_branch_name> origin/<remote_branch_name>
```
***

### References

<https://stackoverflow.com/a/23961231>  
<https://opensource.com/article/18/4/git-tips>

Thanks for reading this article! Feel free to leave your comments and let me know what you think. Please feel free to drop any comment to improve this article.  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com) Have a great day!

  