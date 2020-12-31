---
title: 'Pimp My Git - Git Mergetool'
date: 2017-05-25
#description: "Article description." # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
#codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Continuous Integration
  - Git
  - Linux
  - Windows
tags:
  - linux
  - meld
  - merging

# comment: false # Disable comment if false.
---
I like to work with git on the command line. But in some cases I prefer UI support. For example, solving merge conflicts is such a case. Git has a command `mergetool`, which can open a graphical tool to solve merge conflicts. But before you can use this command, you had to configure it. In this blog post I'd like to show you how to configure mergetool and how to use it.

Configuration
-------------

First at all, open a shell on Linux. On Windows, open Git Bash. Then choose a graphic tool that should support you solving merge conflicts. `git mergetool --tool-help` shows a list which tools are supported on your machine

```bash
sparsick@sparsick-ThinkPad-T430s > git mergetool --tool-help
'git mergetool --tool=<tool>' may be set to one of the following:
               araxis
               kdiff3
               meld

The following tools are valid, but not currently available:
               bc
               bc3
               codecompare
               deltawalker
               diffmerge
               diffuse
               ecmerge
               emerge
               gvimdiff
               gvimdiff2
               gvimdiff3
               opendiff
               p4merge
               tkdiff
               tortoisemerge
               vimdiff
               vimdiff2
               vimdiff3
               winmerge
               xxdiff

Some of the tools listed above only work in a windowed
environment. If run in a terminal-only session, they will fail.
```
This command shows two lists. The first list shows all tools that are supported by git **and** that are available on your machine (in sample, it is `araxis`, `kdiff3` and `meld`). The second list shows that are also supported by git, but they aren't install on your machine. I use `meld` as graphical tool. It's runnable on Windows and Linux. If you haven't install `meld` on your machine, then it's now the right time to do it or choose another tool. We want to set `mergetool` globally for all our repositories.

```bash
sparsick@sparsick-ThinkPad-T430s > git config --global merge.tool meld
sparsick@sparsick-ThinkPad-T430s > git mergetool
No files need merging
```
If `git mergetool` returns more than `No files need merging,` then the path to your graphic tool isn't set in your `$PATH` system variable (The normal case on Windows systems). It's possible to set the path to the graphical tool directly in git.

```bash
sparsick@sparsick-ThinkPad-T430s > git config --global mergetool.meld.path /c/Program\ Files\ \(x86\)/Meld/Meld.exe
```

Bear two important things in mind: `mergetool` is written without a dot between `merge` and `tool` and `meld` is a placeholder for the name of the graphical tool in the above sample. If you use another tool such like `vimdiff`, then the config key is called `mergetool.vimdiff.path`. Now `git mergetool` is ready to use.

Usage
-----

Now I'd like to demonstrate how to use `git mergetool.` It is used in when we have merge conflicts during a merge action. Let's say we want to merge branch `branch1` into `master` and this merge will have some merge conflicts.

```bash
sparsick@sparsick-ThinkPad-T430s > git merge branch1

Auto-merging test
CONFLICT (content): Merge conflict in test
Automatic merge failed; fix conflicts and then commit the result.
```
Now, we want to solve these conflicts with a graphical tool (in the example, it's meld). `git mergetool` on the command line open the graphical tool of our choice.

```bash
sparsick@sparsick-ThinkPad-T430s > git mergetool

Merging:
test

Normal merge conflict for 'test':
{local}: modified file
{remote}: modified file
```
After solving the merge conflicts, the change has to commit.

```bash
sparsick@sparsick-ThinkPad-T430s > git status

On branch master
All conflicts fixed but you are still merging.
(use "git commit" to conclude merge)

Changes to be committed:

modified:   test

Untracked files:
(use "git add <file>..." to include in what will be committed)

test.orig
sparsick@sparsick-ThinkPad-T430s > git commit
```
You can see that we have a new untracked file `test.orig .` This is a backup of the merged file created by `mergetool.` You can configure that this backup should be removed after a successful merge.

```bash

sparsick@sparsick-ThinkPad-T430s > git config --global mergetool.keepBackup false
```
Further files are created when using `git mergetool`:

```bash
sparsick@sparsick-ThinkPad-T430s > git status

On branch master
Untracked files:
(use "git add ..." to include in what will be committed)

test.BACKUP.7344
test.BASE.7344
test.LOCAL.7344
test.REMOTE.7344
```

If only these files are currently untracked, then a `git clean` can help. Otherwise they have to be removed manually.

```bash
sparsick@sparsick-ThinkPad-T430s > git clean -f

Removing test.BACKUP.7344
Removing test.BASE.7344
Removing test.LOCAL.7344
Removing test.REMOTE.734
```

Links
-----

1.  [Meld Homepage](http://meldmerge.org/)
2.  [git mergetool Documentation](https://www.git-scm.com/docs/git-mergetool)
