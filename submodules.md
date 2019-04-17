# Submodules

It often happens that while working on one project, you need to use another project from within it. Perhaps it’s a library that a third party developed or that you’re developing separately and using in multiple parent projects. A common issue arises in these scenarios: you want to be able to treat the two projects as separate yet still be able to use one from within the other.

Git addresses this issue using submodules. Submodules allow you to keep a Git repository as a subdirectory of another Git repository. This lets you clone another repository into your project and keep your commits separate.

## Cloning a repository that contains submodules

If you want to clone a repository including its submodules you can use the ```--recursive``` parameter.

```sh
git clone --recursive [URL to Git repo]
```

If you already have cloned a repository and now want to load it’s submodules you have to use submodule update.

```sh
git submodule update --init
```

if there are nested submodules:

```sh
git submodule update --init --recursive
```

## Adding a submodule

You add a submodule to a Git repository via the ```git submodule add``` command.

```sh
$ git submodule add [URL to Git repo]
```

First you should notice the new ```.gitmodules``` file. This is a configuration file that stores the mapping between the project’s URL and the local subdirectory you’ve pulled it into:

```sh
[submodule "subProject"]
	path = subProject
	url = https://git.server/subProject
```

If you have multiple submodules, you’ll have multiple entries in this file. It’s important to note that this file is version-controlled with your other files, like your .gitignore file. It’s pushed and pulled with the rest of your project. This is how other people who clone this project know where to get the submodule projects from.