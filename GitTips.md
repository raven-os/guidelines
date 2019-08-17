# Git tips

*This is a quick guide to help with git workflow.*

*Do not hesitate to use man pages or your search engine to learn more.*

*If you think something is erronous, or something should be here because it will help, please PR!*

## Bailing out

I put this section at the begining because I think it may help the most :)

If you're having trouble with a merge, rebase, cherry-pick and so on, do not hesitate to use `--abort`.
Ex:
```bash
git rebase --abort
git cherry-pick --abort
git merge --abort
```
You can then restart or try another approache.

## Staging files

Everyone knows `git add`.
Some options can be useful, the most unknown one from experience is `-p` which allows to add specific modifications.

Unstaging files can be done with `git reset`.

Removing files can be done with `git rm`. If you want to keep the file locally use `--cached`.

## Basic commits

```bash
git commit
git commit -a
```
The standart ways to commit. Notice the lack of `-m`.
Usually using `-m` encourages to write long commit titles with no body.

`-a` automatically stages modified files that are tracked before committing.

These kinds of commits are acceptable on any branch, and when doing administrative work on master, are the privileged way of doing things.

## Modifying history

**Never use these on branches on which other branches depend without being sure at 100% that no one may pull this branch**

These operation usually change commit hashes and thus require to force-push (`-f` flag).

This means that someone merging this branch with a previous version will merge the original commits with the modified commits, thus commits may be "doubled".

### Editing commits

#### Amend

```bash
git commit --amend # often used with -a
```
This allows to add the current modifications to the previous commit.

I strongly advise to edit commits in PRs rather than stacking commits to fix mistakes (for example formatting).

#### Rebase

It may happen however that one needs to change a commit that is not the last one:
```bash
git rebase -i <base_branche> # typically master or origin/master
```
This will open a file which contains several lines, each starting with `pick`.
You can replace `pick` with `edit` to change a commit and `drop` to not keep a commit.
You can also edit commit messages.

The hashes of all commits following the first change will change.

### Getting up to date with master

Using `git pull origin master` has the disadvantage that merges don't happen when the branches first conflicts, plus merges sometimes mix several things. Reminder `git pull` is `git fetch` followed by `git merge`.

Using `git fetch origin master` and then `git rebase origin/master` allows to always have your commits be after those of the master branch.

Note that this can be achieved using `git pull --rebase` which will save you some time.

This also means that instead of having merges, you will fix conflicts in the appropriate commits, thus sanitizing the history.

Of course, this requires to force push (see warning above).

## Retrieving commits of another branch

```bash
git cherry-pick <commit-range>
```
Allows to retrieve a range of commits and apply them on the current branche.

This changes their hash, so usually this should only be used salvage work or port security patches.

I recommend not to use this if both branches will have to be merged together, because as when rebasing / amending, the commits will be considered different (see warning about modidying history).

## Handling stashes

### Basics

To store current unstaged changes, use
```bash
git stash [push]
```

To retrieve modifications:
```bash
git stash pop # not apply
```
`pop` also gets rid of the stash if no conflicts happen, and thus avoids having a history of stashes lying around.

To list stashes:
```bash
git stash list
```

To view the last stash:
```bash
git stash show [<stash>] # you can here git diff options here, often -p to view in patch form
```

### Advanced

```bash
git stash push -p
```
Allows to select which modifications go into a hash, really usefull to seperate modifications (same procedure as `git add -p`).


```bash
git stash push -- <file>...
```
Makes a stash only with the specified files.

Finally, the `-m` option allows to name stashes, which can really help hen juggling with a lot of modifications.

## Reseting branches

Haven't done this a lot, but if you need a branch to be like another branch, `git reset --hard` is usually the way to go.

Be sure that is what you really want! On a master branch where you usually don't want to force push (and usually shouldn't even have the rights necessary even as admin), `git revert` is prefered.

## A word on merging

Merging (and connsequently using git pull), should usually only be done when not differing from the remote at all.
*If you fail a merge, remember about `git merge --abort`* and try a different approache (rebasing, reseting the branch to the origin and so on)
