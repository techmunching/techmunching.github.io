---
layout:	post
title:	"Create and manage multiple GitHub accounts"
date:	2019-10-14
categories: [ 'Git' ]
image: 'img/1_kkcqnMF8wuVE-GaXT-r1Gg.jpeg'
author: admin
---

  Managing personal and professional Github Account simultaneously

![](/img/1_kkcqnMF8wuVE-GaXT-r1Gg.jpeg)*Managing Git*

Say you have a personal [GitHub](https://github.com/) account, everything is working perfectly fine. For managing the coexistence of your professional account ([enterprise Github](https://github.com/enterprise)) and your personal account, you would need to have the ability to push and pull to multiple accounts.

Let’s see how!

***

### Table Of Contents

1. Generating the SSH keys
2. Attach the New Key to corresponding GitHub account
3. Registering the new SSH Keys with the ssh-agent
4. Finally Creating the SSH config file
5. Try it Out

***

### Step 1–Generating the SSH keys

Before generating an SSH key, we should always check existing SSH keys:  
*ls -al ~/.ssh*

If *~/.ssh/id\_rsa* is available, we can reuse it, or else we can first generate a key to the default *~/.ssh/id\_rsa* by running:

```shell
 ssh-keygen -t rsa
```

When asked for the location to save the keys, accept the default location by pressing enter. A private key and public key *~/.ssh/id\_rsa.pub* will be created at the default ssh location `~/.ssh/`. Let’s use this default key pair for our personal account.

```shell
cd ~/.ssh  
ssh-keygen -t rsa -b 4096 -C "your_personal_email@domain.com"  
 # save as id_rsa_personal  
ssh-keygen -t rsa -b 4096 -C "your_work_email@domain.com"  

# save as id_rsa_work, you would have two different keys created ~/.ssh/id\_rsa\_personal  
~/.ssh/id\_rsa\_work
```
The files with the .pub extension are the public files that you would add to your GitHub account.
Now you should have something like —

```shell
id_rsa_personal  
id_rsa_personal.pub  
id_rsa_work  
id_rsa_work.pub
```
***

### Step 2 — Attach the New Key to corresponding GitHub account

We already have the SSH public keys ready, and we will ask our GitHub accounts to trust the keys we have created. This is to get rid of the need for typing in the username and password every time you make a Git push.

Copy the public key *pbcopy < ~/.ssh/id\_rsa\_personal.pub* and then log in to your personal GitHub account:

1. Go to *Settings*
2. Select *SSH and GPG keys* from the menu to the left.
3. Click on *New SSH key*, provide a suitable title, and paste the key in the box below
4. Click *Add key* — and you’re done!

> *For the work accounts, use the corresponding public keys (**pbcopy < ~/.ssh/id\_rsa\_work.**pub) and repeat the above steps in your GitHub work accounts.*

***

### Step 3 — Registering the new SSH Keys with the ssh-agent

To use the keys, we have to register them with the *ssh-agent* on our machine. Ensure ssh-agent is running using the command *eval "$(ssh-agent -s)"*.  
Add the keys to the ssh-agent like so:

```shell

ssh-add -D // clears order keys, if any  
ssh-add ~/.ssh/id_rsa_personal  
ssh-add ~/.ssh/id_rsa_work

```
***

### Step 4 — Finally Creating the SSH config file

We’ve done the bulk of the workload, but now we need a way to specify when we wish to push to our personal account, and when we should instead push to our company account. To do so, let’s create a config file.

```shell
touch ~/.ssh/config // Creates the file if not exists  
vim config // Opens the file in VIM editor  
code config // Opens the file in VS code, use any editor
```
If you’re not comfortable with Vim, feel free to open it within any editor of your choice. Paste in the following snippet.

```shell
# Personal account, - the default config  
Host github.com  
 HostName github.com  
 User git  
 Port 443  
 IdentityFile ~/.ssh/id_rsa_personal  
   
# Work account-1  
Host git.corp.<work_url>.com   
 HostName git.corp.<work_url>.com  
 User git  
 Port 443  
 IdentityFile ~/.ssh/id_rsa_work

```
Create the work specific git config (if not already created)

```shell
touch ~/.gitconfig
```

and add following snippet

```shell

....  
....

[user]  
 name = Your Name  
 email = [w](mailto:pranayku@adobe.com)work_email@work.com
 
 [includeIf "gitdir:~/work_folder/"]  
 path = ~/work_folder/.gitconfig
 
 ```
 The above configuration uses [Conditional includes](https://git-scm.com/docs/git-config#_conditional_includes) introduced in git 2.13 to handle multiple configurations.

Alternatively, to add the config name and email, do *git config user.name* and *git config user.email*.

```
 git config user.name "Your Name" // Updates git config user name  
 git config user.email "work_email@work.com"
```

Create the personal specific git config:
```shell
$ nano ~/Personal/.gitconfig
```

and add —
```shell
[user]  
 email = personal_email@gmail.com
```
***

### Step 5— Try it Out

Proceed to authenticate the keys with GitHub using the commands below:

```shell
$ ssh -T github.com  
# Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access. 

$ ssh -T git.corp.work.com  
# Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.
```

Repositories can be cloned using the clone command Git provides:

```shell
git clone git@github.com:personal_account_name/repo_name.git
```
The work repository will require a change to be made with this command:
```
 git clone git@git.corp.work.com:work_user1/repo_name.git
```
***

Thanks for reading this article! Feel free to leave your comments and let us know what you think. Please feel free to drop any comment to improve this article.  
Please check out our [other articles](https://techmunching.com) and [website](https://techmunching.com) Have a great day!

  