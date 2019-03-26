Git Internals
=============

This is where we'll go over the inner workings and implementation of
Git. We found that understanding this information was fundamentally
important to appreciating how useful and powerful Git is.

First, if it isn't yet clear, Git is fundamentally a content-addressable
filesystem with a VCS user interface written on top of it. You'll learn
more about what this means in a bit.

Plumbing and porcelain
----------------------

Git was initially a toolkit for a version control system rather than a
full user-friendly VCS, it has a number of subcommands that do low-level
work and were designed to be chained together UNIX-style or called from
scripts. These commands are generally referred to as Git's "plumbing"
commands, while the more user-friendly commands are called "porcelain"
commands. you'll be dealing mostly with the lower-level plumbing
commands, because they give you access to the inner workings of Git, and
help demonstrate how and why Git does what it does. Many of these
commands aren't meant to be used manually on the command line, but
rather to be used as building blocks for new tools and custom scripts.

When you run `git init` in a new or existing directory, Git creates the
.git directory, which is where almost everything that Git stores and
manipulates is located. If you want to back up or clone your repository,
copying this single directory elsewhere gives you nearly everything you
need. This entire chapter basically deals with what you can see in this
directory. Here's what a newly-initialized .git directory typically
looks like:

``` {.sh}
$ ls -F1
config
description
HEAD
hooks/
info/
objects/
refs/
```

This what you see by default. The description file is used only by the
GitWeb program, so don't worry about it. The config file contains your
project-specific configuration options, and the info directory keeps a
global exclude file for ignored patterns that you don't want to track in
a .gitignore file. The hooks directory contains your client- or
server-side hook scripts.

This leaves four important entries: the HEAD and (yet to be created)
index files, and the objects and refs directories. These are the core
parts of Git. The objects directory stores all the content for your
database, the refs directory stores pointers into commit objects in that
data (branches, tags, remotes and more), the HEAD file points to the
branch you currently have checked out, and the index file is where Git
stores your staging area information. You'll now look at each of these
sections in detail to see how Git operates.

Git Objects
-----------

Git is a content-addressable filesystem. Great. What does that mean? It
means that at the core of Git is a simple key-value data store. What
this means is that you can insert any kind of content into a Git
repository, for which Git will hand you back a unique key you can use
later to retrieve that content.

As a demonstration, let's look at the plumbing command `git hash-object`
which takes some data, stores it in your `.git/objects` directory (the
*object database*), and gives you back the unique keys that now refers
to that data object.

First, you initialize a new Git repository and verify that there is
(predictably) nothing in the objects directory:

``` {.sh}
$ git init
Initialized empty Git repository
$ find .git/objects
.git/objects
.git/objects/info
.git/objects/pack
$ find .git/objects -type f
```

Git has initialized the objects directory and created pack and info
subdirectories in it, but there are no regular files. Now, let's use
`git hash-object` to create a new data object and manually store it in
your new Git database:

``` {.sh}
$ echo 'test content' | git hash-object -w --stdin
d670460b4b4aece5915caf5c68d12f560a9fe3e4
```

If you again examine your objects directory, you can see that it now
contains a file for that new content. This is how Git stores the content
initially as a single file per piece of content, named with the SHA-1
checksum of the content and its header. The subdirectory is named with
the first 2 characters of the SHA-1, and the filename is the remaining
38 characters.

Once you have content in your object database, you can examine that
content with the git cat-file command. This command is sort of a Swiss
army knife for inspecting Git objects. Passing -p to cat-file instructs
the command to first figure out the type of content, then display it
appropriately:

``` {.sh}
$ git cat-file -p d670460b4b4aece5915caf5c68d12f560a9fe3e4
test content
```

Now, you can add content to Git and pull it back out again. You can also
do this with content in files. For example, you can do some simple
version control on a file. First, create a new file and save its
contents in your database:

``` {.sh}
$ echo 'version 1' > test.txt
$ git hash-object -w test.txt
83baae61804e65cc73a7201a7252750c76066a30
```

Then, write some new content to the file, and save it again:

``` {.sh}
$ echo 'version 2' > test.txt
$ git hash-object -w test.txt
1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
```

Your object database now contains both versions of this new file (as
well as the first content you stored there):

``` {.sh}
$ find .git/objects -type f
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
```

At this point, you can delete your local copy of that test.txt file,
then use Git to retrieve, from the object database, either the first
version you saved:

``` {.sh}
$ git cat-file -p 83baae61804e65cc73a7201a7252750c76066a30 > test.txt
$ cat test.txt
version 1
```

or the second version:

``` {.sh}
$ git cat-file -p 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a > test.txt
$ cat test.txt
version 2
```

But remembering the SHA-1 key for each version of your file isn't
practical; plus, you aren't storing the filename in your system --- just
the content. This object type is called a blob. You can have Git tell
you the object type of any object in Git, given its SHA-1 key, with
`git cat-file -t`:

``` {.sh}
$ git cat-file -t 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a
blob
```

Tree Objects
------------

The next type of Git object we'll examine is the tree, which solves the
problem of storing the filename and also allows you to store a group of
files together. Git stores content in a manner similar to a UNIX
filesystem, but a bit simplified. All the content is stored as tree and
blob objects, with trees corresponding to UNIX directory entries and
blobs corresponding more or less to inodes or file contents. A single
tree object contains one or more tree entries, each of which contains a
SHA-1 pointer to a blob or subtree with its associated mode, type, and
filename. For example, the most recent tree in a project may look
something like this in another Git Repo:

``` {.sh}
$ git cat-file -p master^{tree}
100644 blob a906cb2a4a904a152e80877d4088654daad0c859 README
100644 blob 8f94139338f9404f26296befa88755fc2598c289 Rakefile
040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0 lib
```

The master \^{ tree} syntax specifies the tree object that is pointed to
by the last commit on your master branch. Notice that the lib
subdirectory isn't a blob but a pointer to another tree:

``` {.sh}
$ git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b simplegit.rb
```

Conceptually, the data that Git is storing looks something like this:
![Simple version of the Git data
model](Images\Git_Internals\simple_data_model.PNG "Simple version of the Git data model")

You can fairly easily create your own tree. Git normally creates a tree
by taking the state of your staging area or index and writing a series
of tree objects from it. So, to create a tree object, you first have to
set up an index by staging some files. To create an index with a single
entry --- the first version of your test.txt file --- you can use the
plumbing command `git update-index`. You use this command to
artificially add the earlier version of the test.txt file to a new
staging area. You must pass it the `--add` option because the file
doesn't yet exist in your staging area (you don't even have a staging
area set up yet) and `--cacheinfo` because the file you're adding isn't
in your directory but is in your database. Then, you specify the mode,
SHA-1, and filename:

``` {.sh}
$ git update-index --add --cacheinfo 100644 83baae61804e65cc73a7201a7252750c76066a30 test.txt
```

In this case, you're specifying a mode of 100644, which means it's a
normal file. Other options are 100755, which means it's an executable
file; and 120000, which specifies a symbolic link. The mode is taken
from normal UNIX modes but is much less flexible --- these three modes
are the only ones that are valid for files (blobs) in Git (although
other modes are used for directories and submodules). Now, you can use
`git write-tree` to write the staging area out to a tree object. No `-w`
option is needed --- calling this command automatically creates a tree
object from the state of the index if that tree doesn't yet exist:

``` {.sh}
$ git write-tree
d8329fc1cc938780ffdd9f94e0d364e0ea74f579
```

Checking content of tree object using `git cat-file -p` :

``` {.sh}
$ git cat-file -p d8329fc1cc938780ffdd9f94e0d364e0ea74f579
100644 blob 83baae61804e65cc73a7201a7252750c76066a30 test.txt
```

You can also verify that this is a tree object using the git
`cat-file -t` command:

``` {.sh}
$ git cat-file -t d8329fc1cc938780ffdd9f94e0d364e0ea74f579
tree
```

You'll now create a new tree with the second version of test.txt and a
new file as well:

``` {.sh}
$ echo 'new file' > new.txt
$ git update-index --add --cacheinfo 100644 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
$ git update-index --add new.txt
```

Your staging area now has the new version of test.txt as well as the new
file new.txt. Write out that tree (recording the state of the staging
area or index to a tree object) and see what it looks like:

``` {.sh}
$ git write-tree
0155eb4229851634a0f03eb265b69f5a2d56f341
$ git cat-file -p 0155eb4229851634a0f03eb265b69f5a2d56f341
100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
```

Notice that this tree has both file entries and also that the test.txt
SHA-1 is the "version 2" SHA-1 from earlier (1f7a7a). Just for fun,
you'll add the first tree as a subdirectory into this one. You can read
trees into your staging area by calling git read-tree. In this case, you
can read an existing tree into your staging area as a subtree by using
the `--prefix` option with this command:

``` {.sh}
$ git read-tree --prefix=bak d8329fc1cc938780ffdd9f94e0d364e0ea74f579
$ git write-tree
3c4e9cd789d88d8d89c1073707c3585e41b0e614
$ git cat-file -p 3c4e9cd789d88d8d89c1073707c3585e41b0e614
040000 tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579 bak
100644 blob fa49b077972391ad58037050f2a75f74e3671e92 new.txt
100644 blob 1f7a7a472abf3dd9643fd615f6da379c4acb3e3a test.txt
```

If you created a working directory from the new tree you just wrote, you
would get the two files in the top level of the working directory and a
subdirectory named bak that contained the first version of the test.txt
file. You can think of the data that Git contains for these structures
as being like this:

![The content structure of your current Git
data](Images\Git_Internals\content_structure.PNG "The content structure of your current Git data")

Commit Object
-------------

If you've done all of the above, you now have three trees that represent
the different snapshots of your project that you want to track, but the
earlier problem remains: you must remember all three SHA-1 values in
order to recall the snapshots. You also don't have any information about
who saved the snapshots, when they were saved, or why they were saved.
This is the basic information that the commit object stores for you. To
create a commit object, you call `commit-tree` and specify a single tree
SHA-1 and which commit objects, if any, directly preceded it. Start with
the first tree you wrote:

``` {.sh}
$ echo 'first commit' | git commit-tree d8329f
df4e1f23b886d74d90d15b588174484c73f3f34c
```

You will get a different hash value because of different creation time
and author data. Replace commit and tag hashes with your own checksums
further in this chapter. Now you can look at your new commit object with
`git cat-file`:

``` {.sh}
$ git cat-file -p df4e1
tree d8329fc1cc938780ffdd9f94e0d364e0ea74f579
author Yasser FARHAN <yasser.farhan@valeo.com> 1553467111 +0200
committer Yasser FARHAN <yasser.farhan@valeo.com> 1553467111 +0200

first commit
```

The format for a commit object is simple: it specifies the top-level
tree for the snapshot of the project at that point; the author/
committer information (which uses your user.name and user.email
configuration settings and a timestamp); a blank line, and then the
commit message.

Next, you'll write the other two commit objects, each referencing the
commit that came directly before it:

``` {.sh}
$ echo 'second commit' | git commit-tree 0155eb -p df4e1
60a3084097ed1e021621b9c9fe299ed95a3e989e
$ echo 'third commit' | git commit-tree 3c4e9c -p 60a308
050a675d3e6bf948158bc6a1e0e7114cbfc36f18
```

Each of the three commit objects points to one of the three snapshot
trees you created. Oddly enough, you have a real Git history now that
you can view with the `git log` command, if you run it on the last
commit SHA-1:

``` {.sh}
$ git log --stat 050a67
commit 050a675d3e6bf948158bc6a1e0e7114cbfc36f18
Author: Yasser FARHAN <yasser.farhan@valeo.com>
Date:   Mon Mar 25 00:41:05 2019 +0200

    third commit

 bak/test.txt | 1 +
 1 file changed, 1 insertion(+)

commit 60a3084097ed1e021621b9c9fe299ed95a3e989e
Author: Yasser FARHAN <yasser.farhan@valeo.com>
Date:   Mon Mar 25 00:40:43 2019 +0200

    second commit

 new.txt  | 1 +
 test.txt | 2 +-
 2 files changed, 2 insertions(+), 1 deletion(-)

commit df4e1f23b886d74d90d15b588174484c73f3f34c
Author: Yasser FARHAN <yasser.farhan@valeo.com>
Date:   Mon Mar 25 00:38:31 2019 +0200

    first commit

 test.txt | 1 +
 1 file changed, 1 insertion(+)
```

You've just done the low-level operations to build up a Git history
without using any of the front end commands. This is essentially what
Git does when you run the git add and git commit commands  ---   it
stores blobs for the files that have changed, updates the index, writes
out trees, and writes commit objects that reference the top-level trees
and the commits that came immediately before them. These three main Git
objects  ---   the blob, the tree, and the commit  ---   are initially
stored as separate files in your .git/ objects directory. Here are all
the objects in the example directory now, commented with what they
store:

``` {.sh}
$ find .git/objects -type f
.git/objects/01/55eb4229851634a0f03eb265b69f5a2d56f341
.git/objects/05/0a675d3e6bf948158bc6a1e0e7114cbfc36f18
.git/objects/1f/7a7a472abf3dd9643fd615f6da379c4acb3e3a
.git/objects/3c/4e9cd789d88d8d89c1073707c3585e41b0e614
.git/objects/60/a3084097ed1e021621b9c9fe299ed95a3e989e
.git/objects/83/baae61804e65cc73a7201a7252750c76066a30
.git/objects/d6/70460b4b4aece5915caf5c68d12f560a9fe3e4
.git/objects/d8/329fc1cc938780ffdd9f94e0d364e0ea74f579
.git/objects/df/4e1f23b886d74d90d15b588174484c73f3f34c
.git/objects/fa/49b077972391ad58037050f2a75f74e3671e92
```

If you follow all the internal pointers, you get an object graph
something like this:

![All the reachable objects in your Git
directory](Images\Git_Internals\objects_in_git_directory.PNG "All the reachable objects in your Git directory")

Git References
--------------

If you were interested in seeing the history of your repository
reachable from commit, say, 1a410e, you could run something like git log
1a410e to display that history, but you would still have to remember
that 1a410e is the commit you want to use as the starting point for that
history. Instead, it would be easier if you had a file in which you
could store that SHA-1 value under a simple name so you could use that
simple name rather than the raw SHA-1 value.

In Git, these simple names are called "references" or "refs"; you can
find the files that contain those SHA-1 values in the `.git/ refs`
directory. In the current project, this directory contains no files, but
it does contain a simple structure:

``` {.sh}
$ find .git/refs
.git/refs
.git/refs/heads
.git/refs/tags

$ find .git/refs -type f
```

To create a new reference that will help you remember where your latest
commit is, you can technically do something as simple as this:

``` {.sh}
$ echo 050a675d3e6bf948158bc6a1e0e7114cbfc36f18 > .git/refs/heads/master
```

Now, you can use the head reference you just created instead of the
SHA-1 value in your Git commands:

``` {.sh}
$ git log --pretty=oneline master
050a675d3e6bf948158bc6a1e0e7114cbfc36f18 (HEAD -> master) third commit
60a3084097ed1e021621b9c9fe299ed95a3e989e second commit
df4e1f23b886d74d90d15b588174484c73f3f34c first commit
```

You aren't encouraged to directly edit the reference files; instead, Git
provides the safer command `git update-ref` to do this if you want to
update a reference:

``` {.sh}
$ git update-ref refs/heads/master 050a675d3e6bf948158bc6a1e0e7114cbfc36f18
```

That's basically what a branch in Git is: a simple pointer or reference
to the head of a line of work. To create a branch back at the second
commit, you can do this:

``` {.sh}
$ git update-ref refs/heads/test 60a3084
```

Now, your Git database conceptually looks something like this:

![Git directory objects with branch head references
included](Images\Git_Internals\objects_with_refs_included.PNG "Git directory objects with branch head references included")

When you run commands like `git branch <branch>`, Git basically runs
that `update-ref` command to add the SHA-1 of the last commit of the
branch you're on into whatever new reference you want to create.

The HEAD
--------

The question now is, when you run `git branch <branch>`, how does Git
know the SHA-1 of the last commit? The answer is the HEAD file.

The HEAD file is a symbolic reference to the branch you're currently on.
By symbolic reference, we mean that unlike a normal reference, it
doesn't generally contain a SHA-1 value but rather a pointer to another
reference. If you look at the file, you'll normally see something like
this:

``` {.sh}
$ cat .git/HEAD
ref: refs/heads/master
```

If you run `git checkout test`, Git updates the file to look like this:

``` {.sh}
$ cat .git/HEAD
ref: refs/heads/test
```

When you run `git commit`, it creates the commit object, specifying the
parent of that commit object to be whatever SHA-1 value the reference in
HEAD points to.

You can also manually edit this file, but again a safer command exists
to do so: `git symbolic-ref`. You can read the value of your HEAD via
this command:

``` {.sh}
$ git symbolic-ref HEAD
refs/heads/master
```

You can also set the value of HEAD using the same command:

``` {.sh}
$ git symbolic-ref HEAD refs/heads/test
$ cat .git/HEAD
ref: refs/heads/test
```
