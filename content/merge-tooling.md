# Merge and beyond

Git exposes many commands
that can be used to
add the work done in a branch 
into another branch.

For the following demonstration,
you can clone a toy repository 
created on purpose:
```console
$ git clone https://github.com/mmesiti/merge-fu.git
$ cd merge-fu
```
Have a look at the branch structure:
```console
$ git graph # alias for git log --oneline --all --graph --decorate
```
For what follows, 
we do not want to see all the branches all the time,
so we want to define a `git gl` alias *locally*
without the `--all` flag:
```console
$ git config alias.gl --oneline --graph --decorate
```
so that we can use it like this:
```console
$ git gl branch-1 branch-2
```


## Merge

`git merge` is the classic command
that is used join 
(potentially more than 2)
branches together.

It will create an additional commit,
called the *merge commit*.

When Git cannot determine unambiguously
how to merge two versions of a given file,
it will produce a *conflict*.

Solving conflicts requires some practice 
and typically some thought.

Complex conflicts 
can be made easier to understand 
by configuring git to show also 
the version in the [*merge base*](https://www.git-scm.com/docs/git-merge-base)
in addition to the two conflicting versions:

```console
git config --global merge.conflictstyle diff3
```

Using a *merge tool* can also help 
when there are large change sets to merge.
Please refer to the [documentation](https://www.git-scm.com/docs/git-mergetool)
for more information.

### Merge Conflict and abort 
Let us now try to merge `branch-1`
into `branch-2`.
We first need to create 
the local branches 
that match the remote ones:
```console
$ git branch branch-1 origin/branch-1
$ git branch branch-2 origin/branch-2
```
We can check what are the differences
between the two branches:
```console
$ git diff branch-1 branch-2
diff --git a/text-file.txt b/text-file.txt
index b7062f5..ce1f6b8 100644
--- a/text-file.txt
+++ b/text-file.txt
@@ -1,4 +1,4 @@
 1st line
 2nd line
 3rd line
-4th line on branch 1
+4th line on branch 2
```

```{exercise} Predict conflicts
Will the merge between branch-1 and branch-2 cause a conflict?
Why?
::::{solution}
Unfortunately, yes. 
Both versions have appended lines at the end,
and Git cannot determine in which order they need to be.
::::
```

First of all, 
to do the merge
we need to switch to `branch-2`:
```console
$ git switch branch-2
```
We will get a conflict:
```console
$ git merge branch-1
Auto-merging text-file.txt
CONFLICT (content): Merge conflict in text-file.txt
Automatic merge failed; fix conflicts and then commit the result.
```
We can check the content of `text-file.txt`:
```console
$ cat text-file.txt
1st line
2nd line
3rd line
<<<<<<< HEAD
4th line on branch 2
||||||| 874ebe0
4th line
=======
4th line on branch 1
>>>>>>> branch-1
```
If we know how to solve it,
we can modify the file,
stage it and commit.

But what to do in the unhappy situation
where we are not sure how to proceed?
We stop the merge with the command
```console
$ git merge --abort
```
The `--abort` option 
is a useful "handbrake"
that works also with other commands.

## No conflicts, but still wrong 

There are cases where 
a conflictless `git merge` 
can introduce a bug.

For example, 
switch to the branch `python-example`:
```console
$ git switch python-example
```
Check the content of the `example.py` file:
```console
$ cat example.py
def add1(n):
    res = n

    print("This function adds 1 to the input")

    return res
```
This is obviously wrong:
the function is not adding 1.

Fortunately, we have already two possible fixes,
by Alice and Bob.
One is on branch `python-example-fix-1`:
```console
$ git switch python-example-fix-1
$ cat example.py
def add1(n):
    res = n + 1

    print("This function adds 1 to the input")

    return res
```
And another is on branch `pythyon-example-fix-2`:
```console
$ git switch python-example-fix-2
$ cat example.py
def add1(n):
    res = n

    print("This function adds 1 to the input")

    return res + 1
```
They are just one commit away from `python-example`:
```console
$ git gl python-example-fix-1 python-example-fix-2
* 4d8b65f (origin/python-example-fix-2, python-example-fix-2) fix add1
| * 0392b16 (origin/python-example-fix-1, python-example-fix-1) fix add1
|/  
* ff35a6e (HEAD -> python-example, origin/python-example) add python example
* 874ebe0 (origin/main, origin/HEAD, main) First commit
```
Excellent! 
We will merge them both
into `python-example`,
to make everybody feel like
their work is appreciated.
We switch to the `python-example` branch:
```console
$ git switch python-example
Switched to branch 'python-example'
Your branch is up to date with 'origin/python-example'.
```
We merge first `python-example-fix-1`:
```
$ git merge python-example-fix-1 
Updating ff35a6e..0392b16
Fast-forward
 example.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```
This merge is a *fast forward*:
`python-example-fix-1` is a direct descendant of 
the `python-example`
so `python-example` can be just moved forward
without too much thinking.

We then merge `python-example-fix-2`:
```
$ git merge python-example-fix-2
Auto-merging example.py
Merge made by the 'ort' strategy.
 example.py | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```
The file is now **wrong**, though:
```console
$ cat example.py
def add1(n):
    res = n + 1

    print("This function adds 1 to the input")

    return res + 1
```
We are adding 1 twice!

This is obviously a contrived example.
But it shows that:
1. conflicts might be annoying,
   but are actually a good thing;
1. merges should always be checked in some way,
   by a human and/or with an automatic test suite.

To fix this, we can undo the commit 
in one of the ways we have already seen.

:::{exercise} Optional: `revert` a merge commit

When reverting a merge commit, 
it is not clear which is the parent commit
to which we want to revert.

Use the `-m` option
(`--mainline`)
to select the version
you want to revert to.
See the [documentation](https://www.git-scm.com/docs/git-revert#Documentation/git-revert.txt--mparent-number)
for `git revert`.
:::


## Cherry-Pick 

There might be a commit in a branch 
that we want to use,
without merging the whole branch on which it was created.

For this we will consider the branches
`proverbs` and 'good-and-bad-commits':
```console
$ git gl proverbs good-and-bad-commits 
```
The branch that contains our work is `proverbs`,
where we started a collection of popular pieces of wisdom.
Perhaps the branch `good-and-bad-commits`
contains some useful work?
We can check it with `git log -p`,
which will show all the changes 
along with the commit messages:
```console
$ git log --oneline -p proverbs..good-and-bad-commits | cat
9396ffd git is hard!!!!!
diff --git a/wisdom.txt b/wisdom.txt
index b3c8fac..ec51263 100644
--- a/wisdom.txt
+++ b/wisdom.txt
@@ -4,3 +4,6 @@ Early to bed,
 early to rise,
 makes a man wealthy, 
 healthy, and wise.
+
+
+I HATE VERSION CONTROL!
82cfb15 Add proverb
diff --git a/wisdom.txt b/wisdom.txt
index c343ccb..b3c8fac 100644
--- a/wisdom.txt
+++ b/wisdom.txt
@@ -1 +1,6 @@
 # Old Proverbs
+
+Early to bed,
+early to rise,
+makes a man wealthy, 
+healthy, and wise.
```
here `proverbs..good-and-bad-commits`
is a way of specifying the range of commits
above *merge base* on the branch `good-and-bad-commits`.

Once we see the content of each commit,
we become interested in applying 
the second-last commit on `good-and-bad-commits`
to the `proverb` branch.

To do so, we switch to the `proverb` branch
```console
$ git switch proverb
```
and use `git cherry-pick`
with the commit we want to apply
```console
$ git cherry-pick  good-and-bad-commits~
Auto-merging wisdom.txt
CONFLICT (content): Merge conflict in wisdom.txt
error: could not apply 82cfb15... Add proverb
hint: After resolving the conflicts, mark them with
hint: "git add/rm <pathspec>", then run
hint: "git cherry-pick --continue".
hint: You can instead skip this commit with "git cherry-pick --skip".
hint: To abort and get back to the state before "git cherry-pick",
hint: run "git cherry-pick --abort".
```
    
We have a conflict,
but the resolution in this case is trivial.

## Rebase

`git rebase` is an alternative to `git merge` 
that typically leads to a clearer commit history.

In particular:
- an additional merge commit is not necessary
- the commit graph has no bifurcations

The `rebase` command 
will try to reapply 
all the commits on the current branch
on top of another branch
(which will be left untouched),
and then point the current branch
at the last commit.

```{callout} The Golden Rule of Rebase
Do not be rude:
`git rebase` *rewrites history*.
Be very careful when rebasing public branches!
```

### Rebase demo

For this demo we will switch on branch `rebase-me`
```console
$ git switch rebase-me
```
and try to rebase it onto the branch `rebase-onto-this`,
which we need to create locally
from the remote branch,
with this command:
```console
$ git branch rebase-onto-this origin/rebase-onto-this
```
We can have a look
at the branch structure:
```console
$ git gl rebase-me rebase-onto-this
* 3b514df (rebase-onto-this) Add line at end
* e459dcd Add an intermezzo
* a4cc39e (origin/branch-1, branch-1) 2nd commit - on branch-1
| * d30163f (HEAD -> rebase-me) 3rd commit - on branch rebase-me
| * 98f36f0 2nd commit - on branch rebase-me
|/  
* 874ebe0 (origin/main, origin/HEAD, main) First commit
```
We see that:
- there is a bifurcation at `874ebe0`
- our current branch (`rebase-me`)
  has 2 commits above the merge base
- the branch we want to rebase on
  (`rebase-onto-this`)
  has 3 commits above the merge base.

To be able to compare the end result
with the initial situation,
we create a "backup branch" as a bookmark:
```console
$ git branch rebase-me-original rebase-me
```

We now can do the proper rebase.
Make sure we are on the `rebase-me` branch:
```console
$ git branch 
  branch-1
  branch-2
  good-and-bad-commits
  main
  proverbs
* rebase-me
  rebase-me-original
  rebase-onto-this
```
then we invoke the rebase command
to rebase the current branch (`rebase-me`)
onto `rebase-onto-this`:
```console
$ git rebase rebase-onto-this
```
This command will try to apply 
all the commits on the current branch (`rebase-me`)
onto the branch `rebase-onto-this`,
one at a time.
For each commit we might get a conflict,
which is the first thing 
```console
Auto-merging text-file.txt
CONFLICT (content): Merge conflict in text-file.txt
error: could not apply 98f36f0... 2nd commit - on branch rebase-me
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 98f36f0... 2nd commit - on branch rebase-me
```
We can resolve this conflict in the way we please.
```console
$ # edit text-file.txt
```
Once we are done,
we can add our changes:
```console
$ git add text-file.txt
```
and tell rebase to continue to the next commit:
```console
$ git rebase --continue
```
When rebase can automatically merge without commits,
it will not ask for our intervention,
but when there are conflicts it will stop and ask us to solve them,
`git add` the results 
and then use `git rebase --continue`.

After all the commits on the current branch are processed,
we will get a linear commit history for the current branch:
```console
$ git log --oneline
f5b0417 (HEAD -> rebase-me) 3rd commit - on branch rebase-me
3b514df (rebase-onto-this) Add line at end
e459dcd Add an intermezzo
a4cc39e (origin/branch-1, branch-1) 2nd commit - on branch-1
874ebe0 (origin/main, origin/HEAD, main) First commit
```
We can compare the new commit history
with the original position of the branch:
```console
$ git gl rebase-me rebase-me-original
* f5b0417 (HEAD -> rebase-me) 3rd commit - on branch rebase-me
* 3b514df (rebase-onto-this) Add line at end
* e459dcd Add an intermezzo
* a4cc39e (origin/branch-1, branch-1) 2nd commit - on branch-1
| * d30163f (rebase-me-original) 3rd commit - on branch rebase-me
| * 98f36f0 2nd commit - on branch rebase-me
|/  
* 874ebe0 (origin/main, origin/HEAD, main) First commit
```

As `merge` and `cherry-pick`,
`rebase` has a `--abort` option.

If we are not satisfied by the result of the rebase
after it completed,
we can use `git reflog rebase-me` 
to determine the last satisfactory commit,
and use `git reset` to move the branch 
to point there again.

Sometimes, 
the same conflicts will need to be solved
over and over in the same way.
In such a situation, the [`rerere` command](https://git-scm.com/book/en/v2/Git-Tools-Rerere)
(`re`use `re`corded `re`solution)
may come in handy.

## Interactive rebase

`git rebase` has an interactive mode
(that can be entered using the `-i` flag,
or `--interactive`)
that can be used to 
perform complex manipulations of 
the commit history in a range.

It is very powerful,
and it is also used to clean the commit history
of a feature branch 
before making a pull request
(in this case,
there are lower chances for conflicts 
because we rebase 
on a commit that is already 
an ancestor of the current branch,
e.g. `git rebase -i HEAD~3`).

It is possible to perform the following actions
on any commit in the range:
- **pick**: keep it in the history;
- **drop**: dropt it from the history;
- **reword**: change only the commit message
- **squash**: remove the commit, but attribute its changes 
  to the previous picked commit.
- **edit**: change the files and the commit message 
  (even create new commits in the meantime - the opposite of squashing)
- **exec**: pause the rebasing and run a command there (e.g., a test suite) 

More information can be read 
from [the manual](https://git-scm.com/docs/git-rebase#_interactive_mode).

:::{exercise} Interactive rebase on another branch
You can practice interactive rebase with
```console
$ git switch rebase-me
$ git reset reset --hard d30163f 
$ git rebase -i rebase-onto-this
```
:::

