# Git Basics

You typically obtain a Git repository in one of two ways:

1.  You can take a local directory that is currently not under version
    control, and turn it into a Git repository, or
2.  You can `clone` an existing Git repository from elsewhere.

In either case, you end up with a Git repository on your local machine,
ready for work.

## Initializing a Repository in an Existing Directory

If you have a project directory that is currently not under version
control and you want to start controlling it with Git, you first need to
go to that project's directory. If you've never done this, it looks a
little different depending on which system you're running:

for Windows:

```{.sh}
$ cd /my_project
```

and type:

```{.sh}
$ git init
```

This creates a new subdirectory named `.git` that contains all of your
necessary repository files  ---   a Git repository skeleton. At this
point, nothing in your project is tracked yet. (See Git Internals for
more information about exactly what files are contained in the .git
directory you just created.)

If you want to start version-controlling existing files (as opposed to
an empty directory), you should probably begin tracking those files and
do an initial commit. You can accomplish that with a few git add
commands that specify the files you want to track, followed by a git
commit:

```{.sh}
$ git add test.txt
$ git commit -m 'initial project version'
```

We'll go over what these commands do in just a minute. At this point,
you have a Git repository with tracked files and an initial commit.

## Cloning an Existing Repository

If you want to get a copy of an existing Git repository  ---   for
example, a project you'd like to contribute to  ---   the command you
need is `git clone`. If you're familiar with other VCS systems such as
Subversion, you'll notice that the command is `clone` and not
`checkout`. This is an important distinction  ---   instead of getting
just a working copy, Git receives a full copy of nearly all data that
the server has. Every version of every file for the history of the
project is pulled down by default when you run git clone. In fact, if
your server disk gets corrupted, you can often use nearly any of the
clones on any client to set the server back to the state it was in when
it was cloned (you may lose some server-side hooks and such, but all the
versioned data would be there ).

You clone a repository with `git clone < url >`. For example, if you
want to clone the Git linkable library called libgit2, you can do so
like this:

```{.sh}
$ git clone https://github.com/libgit2/libgit2
```

That creates a directory named libgit2, initializes a `.git` directory
inside it, pulls down all the data for that repository, and checks out a
working copy of the latest version. If you go into the new libgit2
directory that was just created, you'll see the project files in there,
ready to be worked on or used.

If you want to clone the repository into a directory named something
other than libgit2, you can specify that as the next command-line
option:

```{.sh}
$ git clone https://github.com/libgit2/libgit2 mylibgit
```

That command does the same thing as the previous one, but the target
directory is called mylibgit.

## Recording Changes to the Repository

At this point, you should have a Git repository on your local machine,
and a checkout or working copy of all of its files in front of you.
Typically, you'll want to start making changes and committing snapshots
of those changes into your repository each time the project reaches a
state you want to record.

Remember that each file in your working directory can be in one of two
states: tracked or untracked. Tracked files are files that were in the
last snapshot; they can be unmodified, modified, or staged. In short,
tracked files are files that Git knows about.

Untracked files are everything else  ---   any files in your working
directory that were not in your last snapshot and are not in your
staging area. When you first clone a repository, all of your files will
be tracked and unmodified because Git just checked them out and you
haven't edited anything.

As you edit files, Git sees them as modified, because you've changed
them since your last commit. As you work, you selectively stage these
modified files and then commit all those staged changes, and the cycle
repeats.

![The lifecycle of the status of your
files](Images/2-Git_Basics/status_life_cycle.PNG)

## Checking the Status of Your Files

The main tool you use to determine which files are in which state is the
git status command. If you run this command directly after a clone, you
should see something like this:

```{.sh}
$ git status
On branch master
Your branch is up-to-date with 'origin/ master'.
nothing to commit, working directory clean
```

This means you have a clean working directory  ---   in other words,
none of your tracked files are modified. Git also doesn't see any
untracked files, or they would be listed here. Finally, the command
tells you which branch you're on and informs you that it has not
diverged from the same branch on the server. For now, that branch is
always "master", which is the default; you won't worry about it here.
Git Branching will go over branches and references in detail.

Let's say you add a new file to your project, a simple README file. If
the file didn't exist before, and you run ```git status```, you see your
untracked file like so:

```{.sh}
$ echo 'My Project' > README
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Untracked files:
    (use "git add < file >..." to include in what will be committed)

        README

nothing added to commit but untracked files present (use "git add" to track)
```

You can see that your new README file is untracked, because it's under
the "Untracked files" heading in your status output. Untracked basically
means that Git sees a file you didn't have in the previous snapshot
(commit); Git won't start including it in your commit snapshots until
you explicitly tell it to do so. It does this so you don't accidentally
begin including generated binary files or other files that you did not
mean to include. You do want to start including README, so let's start
tracking the file.

## Tracking New Files

In order to begin tracking a new file, you use the command ```git add```. To begin tracking the README file, you can run this:

```sh
$ git add README
```

If you run your status command again, you can see that your README file is now tracked and staged to be committed:

```sh
$ git status 
On branch master 
Your branch is up-to-date with 'origin/master'. 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage)

        new file: README
```

You can tell that it’s staged because it’s under the “Changes to be committed” heading. If you commit at this point, the version of the file at the time you ran ```git add``` is what will be in the historical snapshot. You may recall that when you ran ```git init``` earlier, you then ran git add < files >  —   that was to begin tracking files in your directory. The ```git add``` command takes a path name for either a file or a directory; if it’s a directory, the command adds all the files in that directory recursively.

## Staging Modified Files 

Let’s change a file that was already tracked. If you change a previously tracked file called CONTRIBUTING.md and then run your ```git status``` command again, you get something that looks like this:

```sh
$ git status
 On branch master 
 Your branch is up-to-date with 'origin/ master'. 
 Changes to be committed: 
 (use "git reset HEAD < file >..." to unstage) 
 
    new file: 
    
        README 
        
Changes not staged for commit: 
    (use "git add < file >..." to update what will be committed) 
    (use "git checkout -- < file >..." to discard changes in working directory)

    modified: CONTRIBUTING.md
```

The CONTRIBUTING.md file appears under a section named “Changes not staged for commit”  —   which means that a file that is tracked has been modified in the working directory but not yet staged. To stage it, you run the ```git add``` command. ```git add``` is a multipurpose command  —   you use it to begin tracking new files, to stage files, and to do other things like marking merge-conflicted files as resolved. It may be helpful to think of it more as “add precisely this content to the next commit” rather than “add this file to the project”. Let’s run ```git add``` now to stage the CONTRIBUTING.md file, and then run ```git status``` again:

```sh
$ git add CONTRIBUTING.md 
$ git status 
On branch master 
Your branch is up-to-date with 'origin/ master'. 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage) 
        
        new file: README 
        modified: CONTRIBUTING.md
```

Both files are staged and will go into your next commit. At this point, suppose you remember one little change that you want to make in CONTRIBUTING.md before you commit it. You open it again and make that change, and you’re ready to commit. However, let’s run ```git status``` one more time:

```sh
$ git status 
On branch master 
Your branch is up-to-date with 'origin/ master'. 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage) 
    
        new file: README 
        modified: CONTRIBUTING.md 
        
Changes not staged for commit: 
    (use "git add < file >..." to update what will be committed) 
    (use "git checkout -- < file >..." to discard changes in working directory) 
    
        modified: CONTRIBUTING.md
```

What the heck? Now CONTRIBUTING.md is listed as both staged and unstaged. How is that possible? It turns out that Git stages a file exactly as it is when you run the git add command. If you commit now, the version of CONTRIBUTING.md as it was when you last ran the git add command is how it will go into the commit, not the version of the file as it looks in your working directory when you run git commit. If you modify a file after you run git add, you have to run git add again to stage the latest version of the file:

```sh
$ git add CONTRIBUTING.md 
$ git status 
On branch master
Your branch is up-to-date with 'origin/ master'. 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage)
    
        new file: README 
        modified: CONTRIBUTING.md
```
## Committing Your Changes 

Now that your staging area is set up the way you want it, you can commit your changes. Remember that anything that is still unstaged  —   any files you have created or modified that you haven’t run git add on since you edited them  —   won’t go into this commit. They will stay as modified files on your disk. In this case, let’s say that the last time you ran git status, you saw that everything was staged, so you’re ready to commit your changes. The simplest way to commit is to type git commit: 

```sh
$ git commit 
```

Doing so launches your editor of choice :

```sh
# Please enter the commit message for your changes. Lines starting 
# with '#' will be ignored, and an empty message aborts the commit. 
# On branch master 
# Your branch is up-to-date with 'origin/ master'. 
# 
# Changes to be committed: 
#       new file: README 
#       modified: CONTRIBUTING.md 
#
```
You can see that the default commit message contains the latest output of the ```git status``` command commented out and one empty line on top. You can remove these comments and type your commit message, or you can leave them there to help you remember what you’re committing. When you exit the editor, Git creates your commit with that commit message.

Alternatively, you can type your commit message inline with the commit command by specifying it after a ```-m``` flag, like this:

```sh
$ git commit -m "artf182: Fix benchmarks for speed"
[master 463dc4f] Story 182: Fix benchmarks for speed 
    2 files changed, 2 insertions( +) 
    create mode 100644 README
```

Now you’ve created your first commit! You can see that the commit has given you some output about itself: which branch you committed to (master), what SHA-1 checksum the commit has (463dc4f), how many files were changed, and statistics about lines added and removed in the commit. 

Remember that the commit records the snapshot you set up in your staging area. Anything you didn’t stage is still sitting there modified; you can do another commit to add it to your history. Every time you perform a commit, you’re recording a snapshot of your project that you can revert to or compare to later.

## Viewing the Commit History 

After you have created several commits, or if you have cloned a repository with an existing commit history, you’ll probably want to look back to see what has happened. The most basic and powerful tool to do this is the ```git log``` command. 

These examples use a very simple project called “simplegit”. To get the project, run 

```sh
$ git clone https://github.com/schacon/simplegit-progit 
```

When you run git log in this project, you should get output that looks something like this: 

```sh
$ git log 
commit ca82a6dff817ec66f44342007202690a93763949 
Author: Scott Chacon < schacon@ gee-mail.com > 
Date: Mon Mar 17 21: 52: 11 2008 -0700 

    changed the version number 
    
commit 085bb3bcb608e1e8451d4b2432f8ecbe6306e7e7 
Author: Scott Chacon < schacon@ gee-mail.com > 
Date: Sat Mar 15 16: 40: 33 2008 -0700 

    removed unnecessary test 
    
commit a11bef06a3f659402fe7563abf99ad00de2209e6 
Author: Scott Chacon < schacon@ gee-mail.com > 
Date: Sat Mar 15 10: 31: 28 2008 -0700

    first commit
```

By default, with no arguments, ```git log``` lists the commits made in that repository in reverse chronological order  —   that is, the most recent commits show up first. As you can see, this command lists each commit with its SHA-1 checksum, the author’s name and email, the date written, and the commit message. 

## Undoing Things 

At any stage, you may want to undo something. Here, we’ll review a few basic tools for undoing changes that you’ve made. Be careful, because you can’t always undo some of these undos. This is one of the few areas in Git where you may lose some work if you do it wrong. 

One of the common undos takes place when you commit too early and possibly forget to add some files, or you mess up your commit message. If you want to redo that commit, make the additional changes you forgot, stage them, and commit again using the ```--amend``` option: 

```sh
$ git commit --amend 
```

This command takes your staging area and uses it for the commit. If you’ve made no changes since your last commit (for instance, you run this command immediately after your previous commit), then your snapshot will look exactly the same, and all you’ll change is your commit message. 

The same commit-message editor fires up, but it already contains the message of your previous commit. You can edit the message the same as always, but it overwrites your previous commit. 

As an example, if you commit and then realize you forgot to stage the changes in a file you wanted to add to this commit, you can do something like this: 

```sh
$ git commit -m 'initial commit' 
$ git add forgotten_file 
$ git commit --amend 
```

You end up with a single commit —  the second commit replaces the results of the first.

## Unstaging a Staged File 

The next two sections demonstrate how to work with your staging area and working directory changes. The nice part is that the command you use to determine the state of those two areas also reminds you how to undo changes to them. For example, let’s say you’ve changed two files and want to commit them as two separate changes, but you accidentally type ``git add *`` and stage them both. How can you unstage one of the two? The git status command reminds you: 

```sh
$ git add * 
$ git status 
On branch master 
Changes to be committed: 
        (use "git reset HEAD < file >..." to unstage) 
        
            renamed: README.md -> README
            modified: CONTRIBUTING.md 
```

Right below the “Changes to be committed” text, it says use ```git reset HEAD < file >... ```to unstage. So, let’s use that advice to unstage the CONTRIBUTING.md file: 

```sh
$ git reset HEAD CONTRIBUTING.md 
Unstaged changes after reset: 
M CONTRIBUTING.md 
$ git status On branch master 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage)
    
        renamed: README.md -> README 
        
Changes not staged for commit: 
    (use "git add < file >..." to update what will be committed) 
    (use "git checkout -- < file >..." to discard changes in working directory) 
    
        modified: CONTRIBUTING.md
```

The command is a bit strange, but it works. The CONTRIBUTING.md file is modified but once again unstaged.

## Unmodifying a Modified File 

What if you realize that you don’t want to keep your changes to the CONTRIBUTING.md file? How can you easily unmodify it  —   revert it back to what it looked like when you last committed (or initially cloned, or however you got it into your working directory)? Luckily, git status tells you how to do that, too. In the last example output, the unstaged area looks like this: 

```sh
Changes not staged for commit: 
    (use "git add < file >..." to update what will be committed) 
    (use "git checkout -- < file >..." to discard changes in working directory) 
    
        modified: CONTRIBUTING.md 
```

It tells you pretty explicitly how to discard the changes you’ve made. Let’s do what it says: 

```sh
$ git checkout -- CONTRIBUTING.md 
$ git status 
On branch master 
Changes to be committed: 
    (use "git reset HEAD < file >..." to unstage) 
    
        renamed: README.md -> README 
```

You can see that the changes have been reverted.

