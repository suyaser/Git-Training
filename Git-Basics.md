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

