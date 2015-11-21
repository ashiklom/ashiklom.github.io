---
title: Introduction to GitHub
author: Alexey Shiklomanov
date: 2015-09-30
layout: post
---

In the previous tutorial, I went over basics of using Git to do version 
control on your local machine. If you understood all the concepts from that, 
using version control via an online repository host should be cake!

For this tutorial, we'll be using GitHub as a model due to its popularity and 
accessibility. However, the same concepts should be largely applicable to 
GitLab, BitBucket, and any other online version control system that supports 
Git.

# Creating an online repository

To start, browse to [www.github.com](www.github.com) (chances are you're 
already there!) and log in with your username and password. If you don't have 
an account, sign up, it's totally free! 

Your home page on GitHub lives at `github.com/<username>`. From here, you can 
see your existing repositories and a feed of updates to them and other 
repositories you may be following. But since you're probably new to GitHub, 
you won't see anything here yet.

To create a repository, click on the green "+" in the upper right hand corner 
and select "New repository". You will be taken to a page where you pick a 
repository name and have the chance for a few other options. For this 
tutorial, use "test-repo" as your repository name (NOTE that the name cannot 
have spaces). Whether or not you add a "Description" is up to you. For the 
sake of this tutorial, leave the "Initialize this repository with a README" 
box unchecked -- we will be importing an existing repository. Click the green 
"Create Repository" and you're done!

You'll be greeted by instructions on how to proceed, which we'll pretty much 
follow.  The first half of the "...or create a new repository from the command 
line" box should look familiar if you're already familiar with Git basics.  
Basically, all an online repository does is store your commit history (for any 
number of branches) online, so all you really have to know how to do is upload 
("push") and download ("pull") from there.

But first, we need to set up your local repository to "track" (i.e. link up 
to) the online repository.

# Linking your repository and uploading code

Fire up the command line and browse to your test repository. The first thing 
you have to do is to specify a "remote" for the repository -- this indicates 
the target of your pushing and pulling. All git commands for working with 
remotes start with `git remote`. You can learn about all the remote commands 
by entering `git remote --help`.

To add a remote, the syntax is:

```
git remote add <name> <URL>
```

The most common choice for the remote name is `origin`, but you can call it 
anything you want. In fact, if you are pushing and pulling from multiple 
locations (i.e. multiple remotes), you should give each an informative name 
like "github", "johns-fork" (more on "forks" later), "private", and so on. For 
our purposes, let's call it "origin" and link it to the URL of your 
newly-created repository. 

```
git remote add origin https://github.com/<username>/test-repo
```

Note that GitHub repository URLs are simply a combination of your username and 
repository name, which makes them easy to remember.

Now that you're linked up, let's upload all of your hard work! The command for 
uploading is `git push`, and in the future, that's all you'll have to do, but 
for the first push, we have to specify where we're pushing to. This is done by 
adding the `--set-upstream` flag as follows:

```
git push --set-upstream <remote name> <remote branch name>
```

In our case:

```
#> git push --set-upstream origin master
Username for 'https://github.com': ashiklom
Password for 'https://ashiklom@github.com':
Counting objects: 32, done.
Compressing objects: 100% (26/26), done.
Writing objects: 100% (32/32), 2.88 KiB | 0 bytes/s, done.
Total 32 (delta 14), reused 0 (delta 0)
To https://github.com/ashiklom/test-repo
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

When you do this, you will be prompted for your GitHub username and password.  
Since you don't necessarily have to push that often, this may not be a 
problem. However, if you prefer, you can set up authentication through SSH, 
which I won't cover here (see the excellent GitHub guide).

This command does several things. First, it modifies your git configuration 
and links the current branch of the current repository with the specified 
branch on the specified remote. If the branch doesn't exist yet on the remote 
(for instance, in our case), the branch is created.  Then, it checks to see 
how many commits it has to upload since the last push and if these commits can 
be safely merged (i.e.  no merge conflicts whatsoever) with your current work.  
Finally, it updates the remote with all the more recent local commits.  This 
is very efficient for large projects because *only the changes are updated* -- 
if you modify 3 lines of code in a project that has tens of thousands, only 
those 3 lines are transferred. This is part of the beauty of version control.

Note that you have to do this *for every new branch*, because the branch is 
part of the pushing location. If you want to configure a local branch to track 
a remote branch without pushing, the command is:

```
git branch --track <branch> <remote>/<remote branch>
```

Now, let's add a new file and commit it.

```
#> touch new-push
#> git add new-push
#> git commit -m "Added new push file"

[master 60d332a] Added new push file
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 new-push

#> git status

On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working directory clean
```

Note that some new information has been added to `git status`: It now tells 
you that your branch is ahead of the remote "origin" and branch "master" 
(`origin/master`) by one commit. You can add as many commits as you'd like 
without pushing, and `git status` will always helpfully let you know how far 
behind your remote is.

To update the remote with the new commit, we can simply do `git push`. Without 
any additional information, `git push` will update the repository to which it 
was linked by `--set-upstream`. If you'd like, you can specify the remote and 
branch manually (e.g. `git push jeffs-remote devel-branch`), but in most 
cases, it's safer and more convenient to push automatically.

```
#> git push

Username for 'https://github.com': ashiklom
Password for 'https://ashiklom@github.com':
Counting objects: 2, done.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 260 bytes | 0 bytes/s, done.
Total 2 (delta 1), reused 0 (delta 0)
To https://github.com/ashiklom/test-repo
   afed427..60d332a  master -> master
```

# Downloading from an online repository

If you're working on a project alone from a single computer, your local work
will always be at or ahead of your remote. However, oftentimes, you will work 
on multiple computers. There are two scenarios we cover here: (1) You already 
have the repository locally, but it's out of date, or (2) You don't 
have a copy of the repository on your local machine and want to download the 
entire thing from scratch. 

## Updating your local repository (pulling)

First, browse to your `test-repo` on GitHub and make a few edits using the 
built-in editor. Note that GitHub forces you to commit every single one of 
these edits, which is how you should do version control locally as well.

Once you have a few commits, go back to your local repository in the command 
line and run `git pull`. 

```
#> git pull
remote: Counting objects: 5, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 5 (delta 2), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (5/5), done.
From https://github.com/ashiklom/test-repo
   60d332a..2d661b8  master     -> origin/master
Updating 60d332a..2d661b8
Fast-forward
 online1 | 1 +
 online2 | 1 +
 2 files changed, 2 insertions(+)
 create mode 100644 online1
 create mode 100644 online2
```

In my case, I had two new commits, each of which created a new file (`online1` 
and `online2`). The `pull` command is actually a combination of two commands:
`fetch`, which downloads all of the new remote commits into a temporary branch 
and the familiar `merge`, which merges the commits of that temporary branch 
into your current branch. Given this structure, it makes sense that it's 
possible to have merge conflicts when doing a pull -- for instance, if your 
local branch has some commits that are not on the remote branch *and* 
vice-versa.

In general, git operates under the philosophy that **the remote is always 
right**. This makes a lot of sense given that git was developed for 
collaboration (specifically, by Linux founder Linus Torvalds to build Linux 
itself!). Therefore, git actually *will not let you push new commits* until 
your local branch is up-to-date with the remote. To avoid this, **always** 
update your local branch with `git pull` before doing `git push`.

## Downloading from scratch (cloning)

The process for downloading Git repositories is VERY straightforward. The 
command you use is `git clone <URL>`, for instance:

```
#> git clone https://github.com/ashiklom/test-repo
Cloning into 'test-repo'...
remote: Counting objects: 34, done.
remote: Compressing objects: 100% (14/14), done.
remote: Total 34 (delta 15), reused 33 (delta 14), pack-reused 0
Unpacking objects: 100% (34/34), done.
Checking connectivity... done.
```

And that's it! Cloning will download the *entire* commit history through the 
most recent commit, as well as all of the branches (to see all branches, use 
the command `git branch --all`). It also automatically sets up all of the 
tracking information so that you can start pushing and pulling right away!

NOTE however that while you are free to clone and pull from any public 
repository, you can only push to repositories for which you have push access. 

By default, the `clone` command creates a new directory in your current 
directory with the same name as the repository and downloads the files into 
there.  If you want to clone into a different directory, just add the target 
directory onto the end of the cloning command (e.g. `git clone 
https://github.com/ashiklom/test-repo tutorial`).
