# git workflow

The following illustrates our git workflow. Please try to observe it when working on leihs.

![Workflow illustration (the text below explains this in readable form)](https://raw.githubusercontent.com/zhdk/leihs/master/doc/images/git_workflow.png)

There are some basic rules you need to observe:

* You never work directly on master. Only on topic or bugfix branches, and then you merge your branch to master.
* You merge up from master into your branch and resolve conflicts in your branch. Only when you took care of all conflicts, merge that into master.
* Use a squash merge (see below) when merging into master.
* Rewriting published history is forbidden. That means **no rebasing** on branches that have ever been pushed. Rebasing on branches that only you have (local branches) is fine.
* `push --force` is disabled in our repository and will not work. If you need to do a non-fast-forward push, it means you did something wrong.

# Adding a new feature

Branch out from master to a feature branch:

```
git checkout master
git pull
git checkout -b xx_my_new_feature
```

`xx` here are your personal initials, so we know how's responsible to clean up your branch if you leave it lying around for too long after it gets merged.

Assuming master was green when you branched, Work on your branch, adding features, changing stuff, and make sure it is green again.

## Applying fixes from master to your branch

If master has received any commits in the meantime, merge them up to your branch so you get any fixes applied to master:

```
git checkout xx_my_new_feature
git fetch
git merge origin/master
```

Resolve any conflicts and commit them **on your branch**.


## Merging your new feature into master

Your branch will now merge conflict-free onto master, if you've followed the instructions above correctly. So do that:

```
git checkout master
git pull
git merge --squash xx_my_new_feature
```

Make sure to use merge, **not rebase**. We want to keep all history from both branches without rewriting any published history. When your feature branch gets deleted, its history will fold into master anyhow, so don't worry about a cluttered history.

The squash helps roll the entire feature into a single commit.

# Tagging a new release

Whenever master is all green, at the end of a sprint/iteration and when the product owner approves the release, we are ready to release a new version of leihs. We do that by tagging a specific commit:

```
git checkout master
git pull
git tag 3.0.0 f0d2f23e
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
git merge --squash xx_fix_xyz
git push
```

## In an older tag

![Workflow illustration (the text below explains this)](https://raw.githubusercontent.com/zhdk/leihs/master/doc/images/git_workflow_for_fixes.png)

Fixing on a tag is a bit trickier. You have to base off the tag instead of master:

```
git checkout 3.1.0
git checkout -b xx_fix_xyz
```
Now fix the error on your branch and wait for your branch to be green. Then tag the HEAD of your branch as a fixed release. **You will also have to port your fix to master** so that anyone else working on newer topic branches gets the fix.

```
git push origin xx_fix_xyz
git tag 3.1.1
git push --tags
git push origin :xx_fix_xyz
```

Do **never** under any circumstances delete or change existing tags.
