---
title: Introduction to Git
author: Alexey Shiklomanov
date: 2015-09-27
layout: post
comments: true
---

# Why version control?

> An essential aspect of creativity is not being afraid to fail.
> --- Edwin Land

How often do you use the "undo" function on your computer? If you're anything 
like me, it's multiple times a *minute*. In other words, I make mistakes or 
second-guess my decisions almost constantly while working. The fact that my 
computer keeps a history of every action I perform and allows me to instantly 
revert them has not only saved me countless hours but also given me the 
courage to try new things without worrying about causing lingering damage.

Now, imagine if you had finer control over the "undo" feature. For instance, 
what if you want to see a history of everything you've done without actually 
reverting?  Or, what if you wanted to try something new but have quick and 
easy access to an older, working version in case you broke something?  
Moreover, what if you wanted to do any of these things while collaborating on 
a project with multiple people?

Enter **version control**. At its core, version control is a combination of a 
more advanced "undo" feature and a dairy or lab notebook. Whenever you make a 
significant change to whatever you're working on, you save it through your 
version control system along with a note describing what you did and why.  
Eventually, you end up with an annotated history of everything you did to a 
file along with the capability to instantly go back to any point in that 
history. If you're working on a project with multiple people, version control 
also keeps track of who did what and when, so that if something breaks, you 
can quickly and easily follow the paper trail and figure out whom to blame.

If all of this sounds appealing to you, read on! The rest of this page is 
devoted to introducing Git--one of the most popular and powerful version 
control systems in use today.


# Getting started with Git

## Prerequisites

There are multiple graphical user interfaces, application plugins, etc. for 
Git, but I've found that the most robust way to use Git is via the command 
line. After all, every major operating system (yes, even Windows!) has the 
command line, and the git commands for them work exactly the same, so if you 
follow my guide, you will be able to use Git on any computer where it is 
installed. Moreover, if you spend a lot of time working remotely--for 
instance, on a high-performance computer cluster--you will ONLY have the 
command line available to you. And finally, the command line guarantees you 
access to every one of Git's myriad of features, whereas most graphical user 
interfaces restrict you to more common ones.

That being said, to be able to follow this guide, you should have some basic 
knowledge of how to use the command line. In case you don't, here is a quick 
rundown of how to navigate your computer using the command line:

* `cd` -- This is the fundamental navigation tool. It stands for "change 
  directory", where "directory" is the computer science term for "Folder." The 
  usage is simply `cd PATH` where `PATH` is the location of the folder 
  *relative to where you currently are*. For instance, if you start in your 
  `home` folder on OSX/Linux (`C:\Users\username` directory on Windows) and 
  want to navigate to the "Documents" folder, the command is simply `cd 
  Documents` (NOTE that it is case-sensitive). To move up in the folder tree 
  (e.g. to return to the home folder after the previous command), type `cd ..` 
  If you ever get lost, `cd` with no argument will take you back to the home 
  directory (i.e.  where your "Documents", "Pictures", "Downloads", etc.  
  folders probably live).

* `ls` -- This tells you the contents of your current folder (think "list").  
  I instinctively issue `ls` after every `cd` to know what files and 
  directories I'm looking at.

* `pwd` -- "Print working directory". This shows you what directory you are 
  currently in.

These commands will help you navigate the command line, but none of them 
actually do anything. There are more online command line guides out there than 
I could possibly list here, so I'll let you discover those on your own. That 
being said, if you want to learn more about specific ways in which I use the 
command line (and there are many -- I use the command line for pretty much 
every aspect of coding and development), follow the "CLI" section of this 
blog!

## Installation

If you're on a Windows or OSX system, the most straightforward way to install 
git is to download the [GitHub app][https://github.com/download] and then have 
it install the command line tools for you. On Linux systems, there is a good 
chance git is already installed (try running `git --version` and see if it 
throws an error), but if it isn't, you can install using standard repository 
management tools (e.g. `apt-get` on Ubuntu, `yum` on CentOS, `pacman` on 
Arch).

On OSX and Linux, you can then open up Terminal and type `git --version` to 
confirm that Git is installed and working. On Windows, your best bet is to use 
the "Git Shell", packaged with the GitHub installation, although an emulator 
like Cygwin or even Windows' native DOS or Powershell will probably work too.

## Creating a repository

Before integrating Git into your real work, I've found it helpful to learn and 
test out Git's features on simple example repository. That way, you can feel 
free to try different commands without the risk of deleting important 
information, and makes it easier to follow along with my directions.

To begin, create a new folder (in any location you choose). You can call it 
whatver you want -- the name doesn't matter and can be changed at any time.  
Once you've created the folder, open up the command line and navigate to that 
directory. Type `ls` and confirm that the directory is empty. 

The command to initialize a Git repository is `git init`. Running it should 
give the following output:

~~~
Initialized empty Git repository in <directory>/.git
~~~

This will set up the directory for version control by creating a hidden folder 
containing relevant Git information and files. Run this command and then check 
for the existence of the folder by entering `ls -a` (the `a` flag shows all 
directories in the current path, including those that are hidden).  Don't 
worry about the contents of this folder -- 99% of what you do in Git will 
never require you to look in there -- but take comfort in the knowledge that 
everything related to Git's knowledge of the repository is in that folder and 
nowhere else. If for whatever reason you want to stop Git tracking the folder 
but keep your files, you can simply delete this hidden Git folder. 

## Tracking a file

Now that we have a folder tracked by Git, let's create a file for Git to 
track.  Using a plain text editor like Notepad or TextEdit, create a blank 
text file called `file1.txt` in your Git test folder. Confirm that the file is 
there with the `ls` command. Once that's done, enter `git status`. You should 
see the following:

~~~
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	file1.txt

nothing added to commit but untracked files present (use "git add" to track)
~~~

There are three different ways that Git perceives changes: working, staged, 
and committed. To "save" -- or in Git's lingo, "commit" -- a change, you 
perform the following steps:

1. **Make the actual change.** A "change" can be as little or big as you want 
   -- there are basically no restrictions on how small or large it can be, but 
   when in doubt, smaller changes are better than bigger ones because they are 
   easier to backtrace and undo. Any changes you make before performing the 
   subsequent steps are make in the **working** state and are not saved to 
   Git.  In the above example, our change was creating the file `file1.txt`.

2. **Stage the change**. Staged changes are those that are ready to be 
   committed, but have not been committed yet. The reason that all changes are 
   not automatically committed is that you may want to make multiple changes 
   first and then commit each change separately. Or, more often, you may only 
   want to commit changes to certain files and not others.  Staging is 
   typically done with the `git add <filename>` command, so for our example, 
   run the command `git add file1.txt`.

   Run `git status` again and you will see the following helpful information.

   ~~~
   On branch master

   Initial commit

   Changes to be committed:
   (use "git rm --cached <file>..." to unstage)

           new file:   file1.txt

   ~~~

   The change we made -- the addition of `file1.txt` -- will be committed, or 
   permanently saved, the next time we run `git commit`. However, if we change 
   our minds, at this point, we are free to "unstage" that change (by running 
   `git reset HEAD <name of staged file>` -- don't worry about what all of 
   this means yet) and/or add more changes as we see fit. You can use `git 
   add` to stage changes as many times as you want before actually committing.
   
3. **Commit the change**. This is where you make a note describing the change 
   the change you made and why you made it and then save the change to Git's 
   history. Running `git commit` by itself will open up a simple text editor 
   asking you to write a commit message. If you feel comfortable using 
   something like "nano" or "vi" to edit files in terminal, proceed with that; 
   otherwise, a simpler approach is to enter your message directly into the 
   command by running `git commit -m "<your message here>"` (making sure to 
   surround the message in quotes. For our example, enter the following: `git 
   commit -m "Create file1.txt"` and you will see the following summary of the 
   change:

~~~
[master (root-commit) 810e800] Create file1.txt
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1.txt
~~~

*__NOTE__: At this point, Git may ask you for some information if it doesn't 
have it already, such as your name and email.  This is used to store who 
committed each change in the history, which is very useful for collaborations.  
This information is a global setting, so once you set it for the first time on 
your computer, you never have to do it again (unless you want to change it).*

One critical feature of Git is the ability to view your revision history. This 
can be done via the `git log` command, which returns the full list of past 
commits for a repository:

~~~
commit 810e800fb4f35d18f06fd385212b7d1830c0f099
Author: <your name>
Date:   <the current date and time>

    Create file1.txt
~~~

That long string of letters and numbers after "commit" is the automatically 
assigned unique identifier of the commit. When you refer to commits in Git 
commands, you have to use this ID rather than the commit message or anything 
else like that. Fortunately, you don't have to use the entire string -- the 
first 6 characters will do in just about any reasonable case. More on that 
later...

## Editing a file

Our first change created a new file. Now, let's edit it! 

First, for illustrative purposes, run `git status` and you should see the 
following:

~~~
On branch master
nothing to commit, working directory clean
~~~

This is good! It means all of our changes are committed to Git's history. Now, 
fire up your text editor of choice, add some text to your `file1.txt`, and 
save it. Run `git status` again and you will see the following message:

~~~
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   file1.txt

no changes added to commit (use "git add" and/or "git commit -a")
~~~

Git recognizes that the version of `file1.txt` currently in the directory is 
different from the one at the end of one it has saved, so it detects that a
change has been made. To see exactly what has changed, you can use `git diff`, 
which shows all of the unstaged changes made since the last commit:

~~~
diff --git a/file1.txt b/file1.txt
index e69de29..e965047 100644
--- a/file1.txt
+++ b/file1.txt
@@ -0,0 +1 @@
+Hello
~~~

This may look a little bit crypic, but all it's saying is that the line 
"Hello" has been added to the file `file1.txt`. Lines that have been added or 
modified start with `+` while those with deletions start with `-`. The `git 
diff` command has many more powerful uses that you are welcome to explore on 
your own (and that I may address in a later post).

Once you are satisfied with the text you want to add, run `git add file1.txt` 
to stage this change followed by `git commit -m "<message>"` to permanently 
save it. Now, running `git log` shows both the original change and the one we 
just made:

~~~
commit 33b2866809b16fb0e95f48d9c34f01f812ed97a5
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Mon Sep 14 00:06:08 2015 -0400

    Add text to file1.txt

commit 810e800fb4f35d18f06fd385212b7d1830c0f099
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Sun Sep 13 23:27:54 2015 -0400

    Create file1.txt
~~~

## Going back in time

So now that we've been able to generate a revision history with Git, how do we 
use this history? There are actually loads and loads of useful things you can 
do, from rewriting history to grabbing specific files from different points in 
time and much more. In this section, I'm going to cover just one basic aspect: 
Undoing a particular commit.

The command we use to accomplish this is `git revert`, which creates a new 
commit that exactly undoes one specific commit. First, for illustrative 
purposes, I'm going to add a few more things to my history.  Here is my new 
`git log`:

~~~
commit 1b16477bdecec11e9d97e9056a2eede7278709ef
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Fri Sep 18 19:00:38 2015 -0400

    Edit file 2

commit 1e20e7604d28e6a31dd8ce9b0ac8495c1fbe1044
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Fri Sep 18 19:00:22 2015 -0400

    Edit file 1

commit 9d7e4ddf465684ad2f005198590680ccc76d5e6d
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Fri Sep 18 19:00:01 2015 -0400

    Create file2.txt

commit 2642f75f3df7179ec92cd8fa65b0e724dab72758
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Mon Sep 14 00:06:08 2015 -0400

    Add text to file1.txt

commit 8cdb5cc6ef58e952f173fb63cd198314759ccdf5
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Sun Sep 13 23:27:54 2015 -0400

    Create file1.txt

~~~

Now suppose my last edit to File 2 broke something and I wanted to undo that 
change. All I have to do is enter `git revert 1b164`, where the string of 
numbers and letters at the end is just the beginning of the commit hash. This 
is one aspect of Git that is a little user-unfriendly -- these hexidecimal 
commit IDs are impossible to memorize and awkward to input.  Therefore, 
performing operations on commits requires you to master `git log` to quickly 
identify the commits on which you want to operate.  Fortunately, Git has many 
features that limit the necessity of accessing individual commits, such as 
branching, which we'll cover at the end of this tutorial.

Anyway, entering `git revert 1b164` will open a text editor window for you to 
edit a commit message. This is because when Git does a revert, you aren't 
going back in history so much as applying the *exact opposite* of the commit 
of interest. This is Git's way of keeping you honest -- not only do you have a 
record of everything you've done to allow you to undo it, but you keep a 
record of your undoing itself! Feel free to revise the commit message or use 
the default one (usually just `Revert "<Commit message from the commit you 
unid>"`), save it, and quit the editor and you will find yourself restored to 
the previous point in history.

Note that reverting undoes **only the one commit you specify** and will 
**not** undo any commits before or after. This means that you can create 
internal conflicts if you have later commits that depend on the one you've 
reverted. For instance, in my history, I have one commit creating File 2 and a 
later commit revising the same file. If I try to revert only the commit that 
creates the file, I will run into what's called a **merge conflict** because 
my history no longer makes sense. Being the awesome tool that it is, Git will 
helpfully notify you of exactly the location in the file(s) where conflicts 
exist and require you to fix (or at least acknowledge) the conflicts before 
committing. This involves nothing more than fixing (or at least examining) the 
file, staging it via `git add`, and committing the change. However, better 
than having to fix merge conflicts is to avoid them in the first place. In the 
context of undoing, we can accomplish this by reverting a series of commits.
 
In the previous example, we undid just the last commit, but suppose I want to 
undo further back, to the point where I just created File 2. The way to do 
this is to pass a series of commits via `git revert 9d7e4..HEAD`, where 
`9d7e4` is the earliest commit to which we want to revert and `HEAD` refers to 
the current state. When you run this command, you will be taken through all of 
your commits in reverse order and be asked to commit each in turn. For small 
numbers of commits, this is OK, but for larger batches, you may want to accept 
the commit messages automatically, which is done by adding the `--no-edit` 
flag.

Alternatively, you may want to revert to a previous commit, but don't want to 
commit (pun intended) to the change right away. For instance, you may want to 
see if the revert breaks something. To return to a previous point in history 
*without* committing the changes, add the `--no-commit-` or `-n` flag. Using 
our now familiar Git lingo, this will apply the revert to your working 
directory but leave the option to stage and committing the resulting changes 
up to you!

## Branching

Hopefully, the flexibility of the `revert` command has nurtured your 
appreciation for the power of Git, but if that hasn't done the trick, I think 
branching will be an even better sell.

Imagine the following situation: You're working on a project and you get to a 
point where everything works...but not quite as well as you'd like. You have 
some ideas for how to improve the project, but you're not sure whether they'll 
actually work, and you're a little nervous about trying to implement them 
since they might break your current setup. Enter branching, which effectively 
allows you to split your heretofore linear history into multiple "branches" 
and effortlessly switch between them. This is best illustrated with an 
example:

Suppose I have the following `git log`:

~~~
commit a84fda6c9db47d051205a25c583f49c4aa590849
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Mon Sep 21 23:08:23 2015 -0400

    Create file 3

commit 1e942536e28b20a5656ecc26e89e45223ebf85b2
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Mon Sep 21 23:08:09 2015 -0400

    Create file 2

commit 2642f75f3df7179ec92cd8fa65b0e724dab72758
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Mon Sep 14 00:06:08 2015 -0400

    Add text to file1.txt

commit 8cdb5cc6ef58e952f173fb63cd198314759ccdf5
Author: Alexey Shiklomanov (LVM64) <alexey.shiklomanov@gmail.com>
Date:   Sun Sep 13 23:27:54 2015 -0400

    Create file1.txt
~~~

I want to add some new files to this, but I'm a little worried that they won't 
play nicely with my old files. So, rather than risking breaking my current 
setup, I'm going to create a branch called "file-a" as follows: `git branch 
file-a`. If I enter `git branch` with no arguments after doing so, I see the 
following:

~~~
  file-a
* master
~~~

This shows me that I have two branches -- `file-a` and `master` (the default 
branch) -- and that I am currently on the branch `master`. To switch between 
branches, I use the `git checkout <branch name>` command, so in this case, I 
call `git branch file-a`, which outputs:

~~~
Switched to branch 'file-a'
~~~

If I look at my current working directory and history, they are exactly the 
same, which makes sense because I haven't actually "grown" this branch 
anywhere compared to "master". Let's add a file and stage and commit it on 
this new branch:

~~~
#> touch file.a
#> git add file.a
#> git commit -m "Create file.a"

[file-a 95b8fa7] Create file.a
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file.a
~~~

This commit now appears in my Git log, and the file is obviously present in 
the directory. 

~~~
#> ls
file.a  file1.txt  file2.txt
~~~

Now it gets cool. 

Suppose this does break something and I suddenly need to go back to my 
previous version to run something in its working condition. I could undo this 
change with `revert`, but I feel like I've made progress and I want to build 
on it first. All I have to do to go back to the `master` branch is enter `git 
checkout master`:

~~~
Switched to branch `master`
~~~

Now, what does my working directory look like:

~~~
#> ls
file1.txt  file2.txt
~~~

`file.a` isn't there! If you check the Git log, you won't find the commit 
adding `file.a` in there either. But don't fear: `file.a` hasn't been deleted; 
rather, it's stored by Git (in a highly compressed and optimized format) and 
is associated with the branch `file-a`. To go back to it, simply `git checkout 
file-a` again and voila!

~~~
#> ls
file.a  file1.txt  file2.txt
~~~

You can keep adding as many commits as you want to the `file-a` branch, and 
you can have as many branches as you'd like and freely move between them. So, 
if you wanted to try a completely different approach (say `file-c`) starting 
from `master` without losing your work on `file-a`, you totally could! And you 
can have branches branching off of other branches and so on for as complex a 
tree as you'd care to make!

But having branches go off forever isn't the best solution -- what if you find 
that you get to a point on `file-a` where it is objectively better than your 
branching point on `master`? The thing to do here is to *merge* your branch 
with the master branch. This will apply every commit on your `file-a` branch 
to the master branch, bringing both branches fully in sync with each other. In 
our simple example, we first checkout the master branch (`git checkout 
master`) and then `git merge file-a` to merge the `file-a` branch into the 
current `master` branch, giving the following output:

~~~
Updating a84fda6..95b8fa7
Fast-forward
 file.a | 0
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file.a
~~~

If you'd like, you can now delete the "file-a" branch since it is no longer 
required and will clutter your workspace -- the command to do this is `git 
branch -d file-a`. Note that Git will not let you delete an unmerged branch 
that is ahead of its parent in this way. To force the deletion (*be 
careful!*), use the capital `-D` flag.

In the case of a strictly linear history (i.e. every commit on "file-a" is 
chronologically and logically ahead of "master"), merging is always this 
simple. As you use Git more, you will encounter situations where the history 
is not strictly linear; for instance, you may branch off from "master" to 
develop a new feature but then *also* update your "master" branch to implement 
minor bug-fixes or something like that. In such cases, there will be some 
commits on "master" that are not in the history of your other branches. When 
this happens, a merge may not necessarily have the intended consequences and 
will will prompt for a commit, even if there are not explicit conflicts. And 
in some cases, you may have two different versions of the same file on 
branches you are trying to merge, in which case the merge will create a 
conflict (akin to our previous discussion of `git revert`) that you have to 
resolve before completing the merge.

# Conclusion and summary

Those are all the basics you need to know to start effectively using Git 
locally (i.e. without linking to any remote service). Here is a summary of the 
topics we covered:

* **Why use version control?** To keep track of everything you've done and 
  provide you with powerful tools to undo and revise it!
* **Setup**: The recommended way to install Git is by installing the GitHub 
  app and using its menus to set up Git command line tools.
* **Creating a repository:** Go to an existing directory (empty or occupied, 
  it doesn't matter) and run `git init`.
* **Tracking and editing files:** Git moves files between three locations: the 
  working directory, the staging area, and the history. Changes are made in 
  the working directory (edited), moved to the staging area when they are 
  ready to be committed (`git add`, or `git rm` to remove), and then committed 
  to the history (`git commit`).
* **Undoing saved changes:** The `git revert` command undoes a specific commit 
  by creating a *new* commit that is the exact opposite action. To undo 
  multiple commits, pass a series of commits connected by two periods (`..`, 
  i.e. `<earliest commit to undo>..<latest>`). Previous commits are identified 
  by their commit hash, a long string of hexidecimal numbers and letters that 
  can be abbreviated to the shortest unique length. The latest saved commit is 
  also referred to as the `HEAD`.
* **Branching:** Branching is a very powerful and convenient way to make 
  different versions of files between which you can move quickly and easily.  
  To create a branch, enter `git branch <name>`. To switch to another branch, 
  use `git checkout <name>`. Merging a branch transfers all of its commits to 
  the current branch; this is done with `git merge <name>`.

There was a lot of material in this tutorial! Fortunately, once you've 
mastered this one, the next tutorial is a pretty straightforward one (but 
incredibly useful!) on using Git with online "remote" repositories like 
GitHub.
