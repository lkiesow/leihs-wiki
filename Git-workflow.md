# git workflow

The following illustrates our git workflow. Please try to observe it when working on leihs.

![Workflow illustration](https://raw.githubusercontent.com/zhdk/leihs/next/doc/images/git_workflow.png)

# Adding a new feature

Branch out from next to a feature branch:

```
git checkout next
git pull
git checkout -b xx_my_new_feature
```

`xx` here are your personal initials, so we know how's responsible to clean up your branch if you leave it lying around for too long after it gets merged.

Assuming next was green when you branched, Work on your branch, adding features, changing stuff, and make sure it is green again.

## Applying fixes from next to your branch

If next has received any commits in the meantime, merge them up to your branch so you get any fixes applied to next:

```
git checkout xx_my_new_feature
git fetch
git merge origin/next
```

Resolve any conflicts and commit them **on your branch**.


## Merging your new feature into next

Your branch will now merge conflict-free onto next, if you've followed the instructions above correctly. So do that:

```
git checkout next
git pull
git merge --squash xx_my_new_feature
```

Make sure to use merge, **not rebase**. We want to keep all history from both branches without rewriting any published history. When your feature branch gets deleted, its history will fold into next anyhow, so don't worry about a cluttered history.

The squash helps roll the entire feature into a single commit.

# Tagging a new release

Whenever next is all green, at the end of a sprint/iteration, we are ready to release a new version of leihs. We do that by merging next to master, waiting to see if the build is green and then tagging the HEAD of master:

```
git checkout next
git pull
git checkout master
git merge next
git push
```
Now wait for master to be green on CI. When it is, tag it:

```
git checkout master
git tag 3.9.9
git push --tags
```

# Inserting a bugfix

## In the latest released version

If the bug was discovered on the latest currently released version, you fix it on master by creating a new branch off master:

```
git checkout master
git pull
git checkout -b xx_fix_xyz
```

You then fix the bug here and wait for your branch to become green. Once it's green, you merge it into master:

```
git checkout master
git pull
git merge xx_fix_xyz
git push
```
Once master is green as well (it should be), merge the fix up to next:

```
git checkout next
git pull
git merge master
git push
```
That way, anyone working on a branch of next will receive this bugfix when they merge up into their topic branch from next.


## In an older tag

Fixing on a tag is a bit trickier. You have to base off the tag instead of master:

```
git checkout 3.1.1
git checkout -b xx_fix_xyz
```
Now fix the error on your branch. If that bug also appears in master, **you will have to port your fix to master** (using the technique described above). If the bug only affects the tag in question, push your branch, tag the HEAD of your branch as a patch of the existing tag, then delete your bugfix branch:

```
git push origin xx_fix_xyz
git tag 3.1.1-p1
git push --tags
git push origin :xx_fix_xyz
```

Do **never** under any circumstances delete or change existing tags.


