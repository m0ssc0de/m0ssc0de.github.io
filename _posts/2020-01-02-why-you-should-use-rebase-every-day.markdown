---
layout: post
title:  "why you should use rebase every day"
date:   2020-01-02 19:47:07 +0800
categories: git why
tags: git merge rebase why
---

# why you should use rebase every day

Today, let's talk about `merge` and `rebase` as a break time from routine.

There are a lot of documents talking about git `merge` and `rebase`. My goal is not to repeat them, just stating my view of this topic. I appreciate your feedback about anything.

Sometimes we need to make two branches as one. For example, a feature has been finished in a separate branch `feature-branch` from branch `master`. We want to add the new feature to the branch `master`, or add some bug fixing solutions.

The straightforward method is merging branch `feature-branch` to branch `master`. And infrastructures is simple. Switch to branch `master`, `git checkout master`. Merge branch `feature-branch` to branch `master`, `git merge feature-branch`. There will be several different situations now.

As the first assumption, `master` has two commits, `feature-branch` add a new commit. There git logs will look like this.

- `master`: `A <- B`
- `feature-branch`: `A <- B <- C`

> `A <- B` means `A` is the parent commit of commit `B`.

After merging `git merge feature-branch`, they will look like this.

- `master`: `A <- B <- C`
- `feature-branch`: `A <- B <- C`

It's simple and straightforward.

But if there is a new commit `D` after `B`. there will be a little different. Let's see what happens.

- `master`: `A <- B <- D`
- `feature-branch`: `A <- B <- C`

After merging `git merge feature-branch`, they will look like this.

- `master`: 

```
A <- B <- D <- MergeDC
       <- C <-
```

- `feature-branch`: `A <- B <- C`

At branch `master`, there are two new commits, `C` and `MergeDC`.
`C`'s parent commit is `B`. `MergeDC`'s parent commit is `D` and `C`.

So far the result is ok.

If commit `D` and `C` edit the same line, the merge process will be paused. Because there is a conflict now, git doesn't know which one is the final version you want. So we must resolve the conflicts before continuing the process. So do rebase. We just leave how to resolve conflicts in another blog post.

Now let's see why we need rebase.

The first benefit, rebase make git log looks more cleaner. It will make commit `MergeDC` disappear. Let's get the details.

- `master`: `A <- B <- D`
- `feature-branch`: `A <- B <- C`

we now land the branch `feature-branch`. executes `git checkout feature-branch`. And then executes `git rebase master`. The git log will looks like this.

- `master`: `A <- B <- D`
- `feature-branch`: `A <- B <- D <- C`

And then it looks like the first example the simplest one. We can simple merge `feature-branch` to `master`. The branch `master` will looks like this.

- `master`: `A <- B <- D <- C`
- `feature-branch`: `A <- B <- D <- C`

See, the git log looks like one line, and there is no commit have more than one parent commit. So rebase branch `master` before `merge` to it is a good habit.

The second benefit, rebase make git log readable in a safe way. We may make many commits when writing codes. It's too many to read after finish the feature. They could be sorted as one or two meaningful commits.

- `master`: `A <- B`
- `feature-branch`: `A <- B <- C <- D <- E`

- `master`: `A <- B`
- `feature-branch`: `A <- B <- F`

rebase give our change to modify `<- C <- D <- E` to `<- F`. Then the git log look more graceful.

We just leave infrastructures later post.
