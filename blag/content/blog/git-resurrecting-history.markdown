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

This is where [git-fsck(3)][3] can help!

## References ##

[1]: https://www.kernel.org/pub/software/scm/git/docs/git-reflog.html

[2]: https://git-scm.com/book/en/v2/Git-Internals-Git-References

[3]: https://www.kernel.org/pub/software/scm/git/docs/git-fsck.html
