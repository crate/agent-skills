# Contributing

Thank you for your interest in contributing.

This document is a guideline. Don't worry about getting everything perfect.
We are happy to work with you on your contribution.

[Upvoting existing issues](#upvoting-existing-issues), 
[reporting new issues](#repoting-new-issues), or [giving feedback](#tell-us-about-your-experience)
about your experience are the easiest ways to contribute.

We also accept pull requests for changes to the code and to the documentation.

If you have any questions, please reach out via any of our 
[support channels](https://cratedb.com/support).

## Upvoting Issues

An easy way to contribute is to upvote existing issues that are relevant to
you. This will give us a better idea what is important for you and other users.

Please avoid content-less `+1` comments but instead use the emoji reaction to
upvote with a 👍. This allows people to 
[sort issues by reaction](https://github.com/crate/agent-skills/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)
and doesn't spam the maintainers.


## Reporting Issues

Before you report an issue, please 
[search the existing issues](https://github.com/crate/agent-skills/issues?q=is%3Aissue+is%3Aopen+sort%3Areactions-%2B1-desc)
to make sure someone hasn't already reported it.

When reporting a new issue, include as much detail as possible e.g.,

- What you did, what happened, and what you expected to happen
- Steps to reproduce the issue
- Which operating system you're using
- Which version of CrateDB you're running
- Logs or stack traces

## Tell us about your experience

You don't have to create a detailed bug report or request a new feature to make
a valuable contribution. Giving us feedback about your experience with CrateDB is
incredibly valuable as well. Please [get in touch](https://cratedb.com/contact)
with us to tell us what you like and don't like about CrateDB.


## Issue assignments

In general, we will not assign issues to community contributors. If you'd like
to work on an issue you can just state it with a comment, so it's visible to
everyone that you have started working on it, but this shouldn't block any other
potential contributor to work on the same issue. If there are multiple pull
requests opened, we will try to choose the one that is in a better shape. The
reasoning behind this approach, is that we have experienced the case that people
get assigned to an issue without any progress for very long time periods, while
"blocking" other potential contributors from working on that issue.

Note: Of course you're always welcome to ask questions on the issue that would
help you to progress with the implementation.


## Pull Requests

Before we can accept any pull requests, we need you to agree to our Contributor
License Agreement (CLA). If you are contributing as an individual, you must sign
an [Individual Contributor License Agreement](https://app.hellosign.com/s/LP1Ul5Vj)
(ICLA). If you are contributing as an employee of a company and the company 
wants to allow you to contribute your work, then the representative of your 
company must sign a 
[Corporate Contributor License Agreement](https://app.hellosign.com/s/b59e3c0a)
(CCLA).

Note: a CCLA does not remove the need for you to sign you own ICLA as an
individual, to cover both contributions which are owned and those that are not
owned by the company signing the CCLA.

Once that is complete, you should:

- Create an issue on GitHub to let us know that you're working on the issue.
- Use a feature branch and not `main`.
- Rebase your feature branch onto `origin/main` before creating the pull
  request.
- Be descriptive in your PR and commit messages. What is it for? Why is it
  needed? And so on.
- Squash related commits.


### Meaningful Commit Messages

Please choose a meaningful commit message. The commit message is not only
valuable during the review process, but can be helpful for reasoning about
any changes in the code base. For example, IntelliJ's "Annotate" feature,
brings up the commits which introduced the code in a source file. Without
meaningful commit messages, the commit history does not provide any valuable
information.

The subject of the commit message (i.e. first line) should contain a summary
of the changes. Please use the imperative mood. The subject can be prefixed
with "Test: " or "Docs: " to indicate the changes are not primarily to the main
code base. For example:

```
    Add DROP VIEW support to the planner and executor
    Test: Fix flakiness of JoinIntegrationTest
    Docs: Include ON CONFLICT clause in INSERT page
```

See also: https://chris.beams.io/posts/git-commit/

### Updating Your Branch

If new commits have been added to `main` since you created your feature
branch, please do not merge them in to your branch. Instead, rebase your branch::

```shell
git fetch origin
git rebase origin/main
```

This will apply all commits on your feature branch on top of the `main`
branch. If there are conflicts, they can be resolved with `git merge`.
After the conflict has been resolved, use `git rebase --continue` to
continue the rebase process.

### Squashing Minor Commits

Minor commits that only fix typos or rename variables that are related to a
bigger change should be squashed into that commit.

This can be done with the following command::

```shell
git rebase -i origin/main
```

This will open up a text editor where you can annotate your commits.

Generally, you'll want to leave the first commit listed as `pick`, or
change it to `reword` (or `r` for short) if you want to change the commit
message. And then, if you want to squash every subsequent commit, you could
mark them all as `fixup` (or `f` for short).

Once you're done, you can check that it worked by running::

```shell
git log
```

If you're happy with the result, do a **force** push (since you're rewriting history)::

```shell
git push -f
```

