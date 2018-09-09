---
layout: post
title: "Keep Calm and Use Git Reflog"
date: 2017-04-11 07:52:15 +0100
comments: true
categories: ["git"]
authors: ["renaud"]
---

Git is a very powerful tool but it takes a fair amount of time to masterize it. It can be particuarly easy to break or lose things when you amend some commits, resolve conflicts or force push your changes.

In this blog post, I will show you how to use the `git reflog` command to retrieve commits that you thought you lost forever.

<!-- more -->

## Retrieve an amended commit

The first use case I will show is when you amend a commit:

```
$ git init
$ echo "Hello" > hello.txt
$ git add hello.txt && git commit -m "Add hello"
$ echo "Hello!" > hello.txt
$ git add hello.txt && git commit --amend
```

If you think that the first commit is lost, read on.

The `git reflog` command allows you to display all git commands that you performed on your repo and, most importantly, the associated commit hash. In fact, these commits are not lost, they are just not in the tree anymore but you can still check them out, cherry-pick them or do whatever you need to do.

```
$ git reflog
3e6e190 HEAD@{0}: commit (amend): Add hello
51587bc HEAD@{1}: commit (initial): Add hello
$ git diff 51587bc 3e6e190
diff --git a/hello.txt b/hello.txt
index e965047..10ddd6d 100644
--- a/hello.txt
+++ b/hello.txt
@@ -1 +1 @@
-Hello
+Hello!
```

## Recover an erroneous merge

The second scenario is when you rebase a branch on an other one and you have conflicts.

```
$ git checkout -b develop
$ git checkout master
$ echo "Hello, world!" > hello.txt
$ git add hello.txt && git commit -m "Hello, world!"
$ git checkout develop
$ echo "Hello, you!" > hello.txt
$ git add hello.txt && git commit -m "Hello, you!"
$ echo "World!" > world.txt
$ git add world.txt && git commit -m "Add world.txt"
```

This is the resulting tree (note that the hello.txt has been modified on the two branches and there is a conflict):

```
* a5c7ca5 - (HEAD -> develop) Hello, you!
| * 4d016c5 - (master) Hello, world!
|/  
* 3e6e190 - Add hello
```

Unsurprisingly, the `reflog` looks like this:

```
$ git reflog
21c2ce2 HEAD@{0}: commit: Add world.txt
a5c7ca5 HEAD@{1}: commit: Hello, you!
3e6e190 HEAD@{2}: checkout: moving from master to develop
4d016c5 HEAD@{3}: commit: Hello, world!
3e6e190 HEAD@{4}: checkout: moving from develop to master
3e6e190 HEAD@{5}: checkout: moving from master to develop
3e6e190 HEAD@{6}: commit (amend): Add hello
51587bc HEAD@{7}: commit (initial): Add hello
```

Now we rebase `develop` on `master` and we resolve the conflicts.

```
$ git rebase master
// ... resolve merge conflicts
```

The thing is that we made a mistake and took the wrong version of `hello.txt`. Since we did a rebase, we rewrote the history and we could think we lost the "good" version of `hello.txt`.

{% img /images/git-reflog.png %}

In fact, the `reflog` will display a line for each commit you rebased so it is very easy to pick the one where you made a mistake.

```
$ git reflog
ae9d1d8 HEAD@{0}: rebase finished: returning to refs/heads/develop
ae9d1d8 HEAD@{1}: rebase: Add world.txt
4d016c5 HEAD@{2}: rebase: checkout master
21c2ce2 HEAD@{3}: commit: Add world.txt
a5c7ca5 HEAD@{4}: commit: Hello, you!
$ git diff 21c2ce2
diff --git c/hello.txt w/hello.txt
index 624d875..af5626b 100644
--- c/hello.txt
+++ w/hello.txt
@@ -1 +1 @@
-Hello, you!
+Hello, world!
```

If you want to go back and fix it, you just have to check the commit out, delete the current `develop` branch (since it is not in the correct state) and create a new `develop` branch based upon the right commit.

```
$ git checkout 21c2ce2
$ git branch -D develop
$ git checkout -b develop
```

## Conclusion

I think you got the idea, Git keeps everything that you committed. Thus, don't panic if you find yourself in a situation where you erase a previous commit and think about the `git reflog` command to dive into your history and look for the information you thought you lost!
