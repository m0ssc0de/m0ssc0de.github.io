---
layout: post
title: "A practice about rebase"
date:   2020-05-14 08:00:07 +0800
categories: git rebase project
---

Today there is an update in my rust project. New commits were placed in an independent branch. So it's a good chance to record how to use `rebase` in a real project. So let's get the details.

Let's clear the layout. Early commit were placed in `master` branch. New commits were placed in `ws` branch. My final goal is adding the new commits to branch `master`. In addition, merging `ws` to `master`.

Before merging, there are several questions to solve.

1. Does new commits clear?
2. Does two branch conflict?
3. Can we test the final source before real merge?

Solution

1. Maybe some commits are just used for testint or fixing previous commit . We can merge them into one commit. Even delete some useless. With `-i` parameter, `git rebase -i master` will open an interact process. It will let us have chance to modify `git log`. In the processe, it will show you all commits need to be process and the commands will be executed.

For example:
- command `pick` will keep a commit with changes and message.
- command `reword` will have chance to edit the message of a commit.
- command `edit` will have chance to edit the complete commit in `amend` way. It looks like `git commit --amend`, but it could apply to a early commit, not only the last commit.
- command `squash` will combine the commit with previous one with changes and messages.
- command `fixup` will combine the commit to previous one with changes.
- command `exec` will have a chance to run a shell command to do some checks. The shell command will run once. If it returns success, rebase will continue. Otherwise, rebase will stop.
- command `drop` will drop the commit.

2. If some commit have conflict, rebase will stop to wait you make some choice.

- `git checkout --ours <conflict file>` will keep file come from base branch.
- `git checkout --others <conflict file>` will keep file come from base branch.
- or just modify the conflict file

Run `git add <conflict file>`, then `git rebase --continue`.

3. Run some test.

In solution 1, we can add a `exec` command behind the last commit. Let it run some test job. If the job success, rebase finish. If the job failed, come back to edit rebase command. `git rebase --edit-todo` will do rebase from origin.

Or we can just do complex test after finish rebase.

When we finish all process, and make sure no problem, we can merge `ws` branch to `master` branch. Let's get the read process.

## process

Show branch

```shell
$ git branch
master
* ws
```

```shell
git rebase -i master
```

then terminal will show this

```shell
pick 6c5a141 test
pick 4076e32 update
pick 7be75e1 medicine CURD
pick dbaab6c just add ws
pick a0d0a58 splite myws
pick 9398606 hanle text
pick 55b0e36 add proto binary, no test
pick 0dd1902 simple tested
pick 234feaa update proto

# Rebase 9bd0136..234feaa onto 9bd0136 (9 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

The first line is the earliest. The last line is the latest commit. This time, just combine them as one commit. Modify them like this.

```shell
reword 6c5a141 test
fixup 4076e32 update
fixup 7be75e1 medicine CURD
fixup dbaab6c just add ws
fixup a0d0a58 splite myws
fixup 9398606 hanle text
fixup 55b0e36 add proto binary, no test
fixup 0dd1902 simple tested
fixup 234feaa update proto

# Rebase 9bd0136..234feaa onto 9bd0136 (9 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

Just save and quit vi. Then, it will open another vi interface. It's showing `6c5a141` commit's message, `test`. We modify it to `add proto to ws message`. Then save and quit vi.

The last line of terminal show `Successfully rebased and updated refs/heads/ws.`.

That mean we succeed.

Now let's compile code and run the server to check whether the service is ok. Exec `cargo run --bin server`. Do some check. Or exec `cargo test` to run project test.

Let's see `git log` of branch `ws`

```shell
commit 58c72720e70fbfca8cd9fc1cfd4d1cdc4e384239 (HEAD -> ws)
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Wed May 6 21:27:41 2020 +0800

add proto to ws message

commit 9bd013683bf9873ebd0f73ea60a5fe1aeec926f7 (origin/master, origin/HEAD, master)
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Wed May 6 21:03:01 2020 +0800

```
main
```

define struct

commit c0403a7635f6e7c238c2000f8a3ebeae16d76bf0
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Thu Apr 30 17:22:07 2020 +0800

update env shell
...
```

If everything ok, let's move to branch `master`. `git checkout master`.

And merge the `ws` to `master`. `git merge master`.

And show git log.

```shell
$ git checkout master
Switched to branch `master`
Your branch is up to date with 'origin/master'.
$ git merge ws
Updating 9bd0136..58c7272
Fast-forward
.env | 1 +
Cargo.lock | 532 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
Cargo.toml | 26 +++++++++-
build.rs | 4 ++
dataflow.md | 18 +++++++
diesel.toml | 2 +
migrations/.gitkeep | 0
migrations/2020-05-06-144602_create_posts/down.sql | 2 +
migrations/2020-05-06-144602_create_posts/up.sql | 8 ++++
src/db.rs | 57 ++++++++++++++++++++++
src/db/models.rs | 18 +++++++
src/db/schema.rs | 9 ++++
src/items.proto | 69 +++++++++++++++++++++++++++
src/main.rs | 54 ++++++++++++++++-----
src/myws.rs | 100 +++++++++++++++++++++++++++++++++++++++
src/pmodels.rs | 45 ++++++++++++++++++
test.db | Bin 0 -> 5120 bytes
17 files changed, 931 insertions(+), 14 deletions(-)
create mode 100644 .env
create mode 100644 build.rs
create mode 100644 dataflow.md
create mode 100644 diesel.toml
create mode 100644 migrations/.gitkeep
create mode 100644 migrations/2020-05-06-144602_create_posts/down.sql
create mode 100644 migrations/2020-05-06-144602_create_posts/up.sql
create mode 100644 src/db.rs
create mode 100644 src/db/models.rs
create mode 100644 src/db/schema.rs
create mode 100644 src/items.proto
create mode 100644 src/myws.rs
create mode 100644 src/pmodels.rs
create mode 100644 test.db
```

```shell
commit 58c72720e70fbfca8cd9fc1cfd4d1cdc4e384239 (HEAD -> master, ws)
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Wed May 6 21:27:41 2020 +0800

add proto to ws message

commit 9bd013683bf9873ebd0f73ea60a5fe1aeec926f7 (origin/master, origin/HEAD)
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Wed May 6 21:03:01 2020 +0800

define struct

commit c0403a7635f6e7c238c2000f8a3ebeae16d76bf0
Author: m0ssc0de <hi.paul.q@gmail.com>
Date: Thu Apr 30 17:22:07 2020 +0800

update env shell
...
```

Now we can see branch `master` is same with branch `ws`. And we could push it to remote at anytime.