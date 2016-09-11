---
title: "Git: Resurrecting History"
description: ""
tags:
  - "Git"
  - "Tips and Tricks"
  - "How-to"
  - "zData"
date: "2016-09-07"
draft: true
categories:
  - "Development"
slug: "git-resurrecting-history"
---

We all make mistakes. They are inevitable. We must accept that we make them and
move on. But making mistakes in Git seems to be overly complex and most will
simply result to cloning anew and copying the working tree (or some subset) and
moving on. This, to me, however, seems like a waste of bandwidth as most issues
resulting in broken history are in fact quite easy to resolve once the
necessary tools are known.

## Git Reflog ##

> Reference logs or "reflogs", record when the tips of branches and other
> references were updated in the local repository.
--[git-reflog(1)][1]

That is, the reference log is the (meta)log of the actions against branches
(tips) and other [references][2]. Everytime we commit, merge, change branches,
or perform _any_ action that might alter the commit a reference points to, this
change is stored in the reflog of the current repository. For a freshly cloned
repository, the reflog will be quite boring, e.g., a single entry for the
initial clone.

However, after working on a project for while, the reflog will have quite the
history of actions performed.

For example, here is the first 24 lines of the reflog for this blog:

    a1bbd00 HEAD@{0}: checkout: moving from master to git_resurrection
    a1bbd00 HEAD@{1}: commit: Update paths of SSL certificate and key
    d7fd8f8 HEAD@{2}: commit: Add all targets to phony
    f639cbe HEAD@{3}: commit: Add phony target list
    8f3bba4 HEAD@{4}: commit: Add build to deploy dependency
    5331695 HEAD@{5}: merge elixir_releases: Fast-forward
    1a27df5 HEAD@{6}: checkout: moving from elixir_functional_fib to master
    61f755b HEAD@{7}: checkout: moving from master to elixir_functional_fib
    1a27df5 HEAD@{8}: checkout: moving from elixir_releases to master
    5331695 HEAD@{9}: rebase -i (finish): returning to refs/heads/elixir_releases
    5331695 HEAD@{10}: rebase -i (squash): Add Elixir OTP Releases Post
    07f3995 HEAD@{11}: rebase -i (squash): # This is a combination of 4 commits.
    9b7bc7b HEAD@{12}: rebase -i (squash): # This is a combination of 3 commits.
    06414a7 HEAD@{13}: rebase -i (squash): # This is a combination of 2 commits.
    cb59962 HEAD@{14}: rebase -i (start): checkout HEAD~5
    bf8836f HEAD@{15}: commit: WIP: elixir otp releases
    34bc98a HEAD@{16}: commit: WIP: update ends
    00fc016 HEAD@{17}: commit: WIP: elixir otp releases
    e859353 HEAD@{18}: commit: WIP: elixir otp release post
    cb59962 HEAD@{19}: commit: WIP: elixir otp releases post
    1a27df5 HEAD@{20}: checkout: moving from master to elixir_releases
    1a27df5 HEAD@{21}: checkout: moving from elixir_functional_fib to master
    61f755b HEAD@{22}: commit: WIP: some post about fib
    4137e6e HEAD@{23}: checkout: moving from master to elixir_functional_fib

The first column is the commit SHA-1 that is the _result_ of the action, the
second column provides a shortcut reference that can be used anywhere a regular
reference can be, the 3rd column is the action, e.g., `checkout`, `commit`,
`merge`, ect., and a short description of the action. In the case of commits,
the description text will be the summary line of the commit message.

From the reflog, we can see I've recently made a branch for this post, before
that, I made several commits against the `master` branch, and before that, I
performed a fast-forward merge of the local `elixir_releases` branch into the
`master` branch. Ect.

This is some pretty powerful information for digging into the history of the
repository. The reflog is indispensible for working out how to recover the lost
changes of our mistakes.

## Git Fsck ##

[git-reflog(1)][1] is a very important tool, but another way histroy can be
lost is by becoming "unreachable".

This is where [git-fsck(1)][3] can help! [git-fsck(1)][3] searches the Git
object store, and will report objects that are dangling or unreachable from a
named reference. This way, we can find commits, or even blobs, that have been
lost to us because they do not exist in the directed acyclic graph (DAG) of
Git, but _do_ exist in the object store itself.

For example, running `git fsck` on this repository yields the following output:

    ± git fsck
    Checking object directories: 100% (256/256), done.
    Checking objects: 100% (150/150), done.
    dangling commit 16f6063abde9dcd8279fb2a7ddd4998aaf44acc7

Now, if we add another option, namely, `--unreachable`, we get the following:

    ± git fsck --unreachable
    unreachable blob 20c1e21948ab5d9553c11fa8a7230d73055c207e
    unreachable commit 16f6063abde9dcd8279fb2a7ddd4998aaf44acc7
    unreachable commit 41a324739bc3f1d265ecc474c58256e3a4ad4982
    unreachable blob c4131dc6d091b1c16943554fa2396f5d405e8537

Furthermore, objects listed in the reflog are considered "reachable", but may
be still eluding our search. Adding `--no-reflogs` to [git-fsck(1)][3] can help
make these objects more visible:

    ± git fsck --unreachable --no-reflogs
    unreachable commit 00fc0164a78fe6b46e56781d434fdbb893f11534
    unreachable blob 18a484273f75e4a3dcac75cb5229a614f6090be0
    unreachable commit 1cdc30ebd6ebbaba4a8c28fb35457a8d5cb4326f
    unreachable blob 27c4af632030e3d794181024fba120c6db44eef5
    unreachable commit 31a0e98166bc48bf1f725a657e27632c99568da0
    unreachable commit 34bc98ae27f3db69df82b186cf2ef8a86b42ea12
    unreachable commit 8f08be163f185dd130a86d67daf61639632c4e20
    unreachable commit bf8836f2e435ee241ebe53f0eae4ee98bd887082
    unreachable commit 06414a75d58cee81fb2035b8af45a543c6bb09ef
    unreachable blob 1f853af2881919bc62321b536bfc0de6e9602db6
    unreachable blob 20c1e21948ab5d9553c11fa8a7230d73055c207e
    unreachable commit 54cd8b9b5c58409ce3f509e74d5a7a7ac4a73309
    unreachable commit a9693871e765355b6d9a57a612a76f454b177da0
    unreachable commit ad45856329ff97bd35ac17325952c21e53d51b28
    unreachable blob b8154e42d08b74ae6b9817e12b7764d55760c86e
    unreachable commit cb599620e2d364e2ab44ada45f16df05c5fe3f51
    unreachable commit e859353ddc681177141d84a0053b9b8ecad1151e
    unreachable blob fed50bb1d7c749767de7589cc8ef0acf8caf8226
    unreachable blob 056a7e48130d8d22227367ae9753cb5c9afe2d39
    unreachable commit 16f6063abde9dcd8279fb2a7ddd4998aaf44acc7
    unreachable commit 54def8ee3ea0c7043767185e5900480d24ddb351
    unreachable commit 65d2a1553e3c1dd745afa318135a5957e50dd6ef
    unreachable commit 741afdc2f13e76bd0c48e1df7419b37e57733de3
    unreachable commit 7bb6b449ced0493f2d3cc975157aefa84b082e04
    unreachable commit 7e067ad694538a410f98732ce3052546aadc0240
    unreachable commit 809e9d1f131f54701325357199643505773f5d25
    unreachable blob 8802d6dcac8b14399ca4082987a76be4b179333c
    unreachable blob 8b82ffa1eb05ef3306ab62e1120f77a80a887d94
    unreachable commit 9af67536e6852fe928934ba0950809597d73a173
    unreachable blob b23eefdac6b2056e25c748679958179bdbd8f81f
    unreachable blob b66ef50f82242ec929141cf3246278c6160e230a
    unreachable blob c2fa5a98fe1010a1255f032ba34a612e404c7062
    unreachable blob dd42939b3f6cf542064eb011b74749195c951957
    unreachable commit 07f39952cd161438ff4b208b6cb10b287881db85
    unreachable blob 1c0327c6a73923e932eb4f4bf877f660bd13a7b0
    unreachable commit 41a324739bc3f1d265ecc474c58256e3a4ad4982
    unreachable commit 74671b411e2cf1209bc681f0349e24ef7fe00f19
    unreachable commit 9437cbb0500b22a57a62e2cf0a512b1b56ce6a96
    unreachable commit 9a0f5f8c63c184cd5082f27dbe513b3e683bc1ad
    unreachable commit 9b7bc7bf0f01a84621e23bfa02e0a09f63da1747
    unreachable commit bce7c8dbcc56e6935015a5fb2c74224bb8d9f768
    unreachable blob c4131dc6d091b1c16943554fa2396f5d405e8537
    unreachable blob c69782e19aee6d89de4f6bcf9ed14813f72c8c10
    unreachable blob d79fb0b95796290c33d6f3dee004235dad7d8893
    unreachable commit dabb01b3df1371602f3f0689d25359597db54423
    unreachable blob ec2ba85be58685070a44727bc2591b9a32eb6457

Using these hashes, one could inspect them using other [familiar tools][4],
namely, [git-show(1)][5] and [git-cat-file(1)][6] to figure out if these are
worth resurrecting or are in fact the objects we want to resurrect.

## Resurrection Example ##

Now that we have our tools, let's examine a situation where a change to the
history was made that needs to be corrected: squashed public commits.

Let's say we have been working on some feature in our branch, but have somehow
found out that we have squashed too many commits then we intended, changing the
history and making our changes non-pushable. How might we solve this?

To make this example more concrete, let's run a few commands that put us into
this problem:

    $ cd $(mktemp -d)
    $ git init foobar
    $ cd foobar
    ± touch foo
    ± git add foo
    ± git commit -m 'initial commit'
    ± touch bar
    ± git add bar
    ± git commit -m 'add bar'
    ± touch foobar
    ± git add foobar
    ± git commit -m 'add foobar'
    ± echo 1 >> foo
    ± git commit -a -m 'update foo: add 1'

After this, we should have a log similar to the following:

    ± git log --oneline
    f3fb125 update foo: add 1
    976ad96 add foobar
    113f412 add bar
    7e5b674 initial commit

Now, let's assume all of these commits are public except the last one,
`f3fb125` in this case.

Let's perform a rebase, but instead will be squashing in the previous two
commits, `976ad96` and `113f412`.

    ± git rebase -i HEAD~3
    (change the `pick` in the last two commits to `fixup`)
    ± git log --online
    883934d add bar
    7e5b674 initial commit
    ± git show --stat
    commit 883934d0d3f11cdc596f5e6e11185f3e04b6a877
    Author: kballou <kballou@devnulllabs.io>
    Date:   Thu Sep 8 11:58:46 2016 -0600

        add bar

     bar    | 0
     foo    | 1 +
     foobar | 0
     3 files changed, 1 insertion(+)

Here, `883934d` has the last three commits rolled into it, which is clearly not
what we want.

How do we return to the original state?

### Solution ###

Correcting the state in this repository, as it turns out, is actually quite
simple.

Let's look at the reflog now:

   ± git reflog
    883934d HEAD@{0}: rebase -i (finish): returning to refs/heads/master
    883934d HEAD@{1}: rebase -i (fixup): add bar
    9ef31d7 HEAD@{2}: rebase -i (fixup): # This is a combination of 2 commits.
    113f412 HEAD@{3}: rebase -i (start): checkout HEAD~3
    f3fb125 HEAD@{4}: commit: update foo: add 1
    976ad96 HEAD@{5}: commit: add foobar
    113f412 HEAD@{6}: commit: add bar
    7e5b674 HEAD@{7}: commit (initial): initial commit

At `HEAD@{3}`, we see the start of the rebase operation, and at `HEAD@{4}` we
see the original HEAD, before the rebase. Therefore, we can use the
[git-reset(1)][7] command to reset the branch back to the previous state:

    ± git reset --hard HEAD@{4}
    HEAD is now at f3fb125 update foo: add 1
    ± git log --oneline
    f3fb125 update foo: add 1
    976ad96 add foobar
    113f412 add bar
    7e5b674 initial commit

We are now back where we were.

Another way this can be solved is by noticing that [git-rebase(1)][8] (and
other tools within Git) writes the previous `HEAD` commit into a file under the
`./.git` folder called `ORIG_HEAD`. Now that we have done a `git reset --hard`,
the `./.git/ORIG_HEAD` file points to the squashed commit:

    ± cat .git/ORIG_HEAD
    883934d0d3f11cdc596f5e6e11185f3e04b6a877

## Another Example Resurrection ##

For another example, let's examine when we create a branch and change the
parent commit of the branch point.

We will start with some commands that create and initialize the repository into
the initial state, that is, before any mistakes are made:

    $ cd $(mktemp -d)
    $ git init foobar
    $ cd foobar
    ± touch foo
    ± git add foo
    ± git commit -m 'initial commit'
    ± touch bar
    ± git add bar
    ± git commit -m 'add bar'
    ± echo 1 >> foo
    ± git commit -am 'update foo: add 1'
    ± git checkout -b topic/foobar
    ± echo 1 >> bar
    ± git commit -am 'update bar: add 1'

From here, our "oneline" log should look similar to the following:

    ± git log --oneline
    3de2659 update bar: add 1
    5e6dd5f update foo: add 1
    9640abb add bar
    31d2347 initial commit

Furthermore, here is an image that describes the state of the repository.

{{< figure src="/media/git-repo-state-1.svg"
    alt="Example Repository State 1" >}}

Next, we will create a few more commits, but instead of doing things properly,
we are going to (intentionally) make a mistake. We will merge our
`topic/foobar` branch into `master`, create a new file, `foobar`, and create a
branch, `topic/bad`, from `topic/foobar`. In the `topic/bad` branch, we will
create a new commit, but then we will rebase the two previous commits.

Let's being issuing commands against our repository:

    ± git checkout master
    ± git merge --ff-only topic/foobar
    ± touch foobar
    ± git add foobar
    ± git commit -m 'add foobar'
    ± git checkout -b topic/bad topic/foobar
    ± echo 2 >> foo
    ± git commit -am 'update foo: add 2'
    ± echo 2 >> bar
    ± git commit -am 'update bar: add 2'

Thusly, our repository should look similar to the following image:

{{< figure src="/media/git-repo-state-2.svg"
    alt="Example Repository State 2" >}}

Now, for the mistake:

    ± git rebase -i HEAD~3
    (squash the previous commits)

This should result in a repository that looks like the following:

{{< figure src="/media/git-repo-state-3.svg"
    alt="Example Repository State 3" >}}

Assuming we didn't recognize the mistake, we might attempt to merge the branch:

    ± git checkout master
    ± git merge --ff-only topic/bad
    fatal: Not possible to fast-forward, aborting.

Well, of course, the `master` branch is ahead by one commit, and the
`topic/bad` branch is "behind" by two.

We can see this be viewing the logs when going from `master` to `topic/bad` and
then vice-versa:

    ± git log --oneline master..topic/bad
    3b71666 update bar: add 1
    ± git log --oneline topic/bad..master
    7387d60 add foobar
    3de2659 update bar: add 1

But another issue emerges from viewing these log outputs from our mistake
ignorant brains: two of the commits look the same.

Yep, it looks like we have not only combined two of our the changes from
`topic/bad` but we combined them with a commit that was _already_ merged into
the `master` branch. Assuming `master` is a stable and branchable branch, we
will not be able to simply rebase one way and return, the commits are too
intermingled

### Solutions ###

One way we can fix this is to simply not care. But that's not what we are
about: we like clean history, this situation is clearly not clean!

Therefore, we will have to return the `topic/bad` branch to a cleaner state
before continuing with merging the work done in the branch.

Let's start with the reflog:

    ± git reflog
    7387d60 HEAD@{0}: checkout: moving from topic/bad to master
    3b71666 HEAD@{1}: rebase -i (finish): returning to refs/heads/topic/bad
    3b71666 HEAD@{2}: rebase -i (fixup): update bar: add 1
    4cc10e9 HEAD@{3}: rebase -i (fixup): # This is a combination of 2 commits.
    3de2659 HEAD@{4}: rebase -i (start): checkout HEAD~3
    7647f9c HEAD@{5}: commit: update bar: add 2
    4babfe7 HEAD@{5}: commit: update foo: add 2
    3de2659 HEAD@{6}: checkout: moving from master to topic/bad
    7387d60 HEAD@{7}: commit: add foobar
    3de2659 HEAD@{8}: checkout: moving from topic/bad to master
    3de2659 HEAD@{9}: checkout: moving from master to topic/bad
    3de2659 HEAD@{10}: merge topic/foobar: Fast-forward
    5e6dd5f HEAD@{11}: checkout: moving from topic/foobar to master
    3de2659 HEAD@{12}: commit: update bar: add 1
    5e6dd5f HEAD@{13}: checkout: moving from master to topic/foobar
    5e6dd5f HEAD@{14}: commit: update foo: add 1
    9640abb HEAD@{15}: commit: add bar
    31d2347 HEAD@{16}: commit (initial): initial commit

Examining `HEAD@{5}` we will see the commit of `topic/bad` _before_ we
attempted to rebase the three commits. If we start there, we may be able to
salvage the history.

    ± git checkout topic/bad
    ± git reset --hard 7647f9c
    ± git log --oneline
    7647f9c update bar: add 2
    4babfe7 update foo: add 2
    3de2659 update bar: add 1
    5e6dd5f update foo: add 1
    9640abb add bar
    31d2347 initial commit

Perfect, we are back to the state of the branch as seen in the following image:

{{< figure src="/media/git-repo-state-2.svg"
    alt="Example Repository State Before Mistake" >}}

## References ##

[1]: https://www.kernel.org/pub/software/scm/git/docs/git-reflog.html

[2]: https://git-scm.com/book/en/v2/Git-Internals-Git-References

[3]: https://www.kernel.org/pub/software/scm/git/docs/git-fsck.html

[4]: https://kennyballou.com/blog/2016/01/git-in-reverse/

[5]: https://www.kernel.org/pub/software/scm/git/docs/git-show.html

[6]: https://www.kernel.org/pub/software/scm/git/docs/git-cat-file.html

[7]: https://www.kernel.org/pub/software/scm/git/docs/git-reset.html

[8]: https://www.kernel.org/pub/software/scm/git/docs/git-rebase.html
