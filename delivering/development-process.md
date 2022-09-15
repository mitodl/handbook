---
parent: How We Deliver
---
# Development Process
An approximate workflow guide to hacking on software at the MIT Office of Digital Learning.

# Purpose
Every team of developers has its own particular approach to writing, reviewing, and deploying code.
This guide aims to capture MIT ODL's way of doing things. It will be (hopefully) useful as a
step-by-step guide for new developers.

# Tools

- git
- Zenhub Chrome extension (https://www.zenhub.io/)
  - This extension adds the **Boards** and **Burndown** tabs to GitHub repos. We use it in the way
  that you might use JIRA or Trello. The **Boards** tab is particularly important as it adds a
  'swim lane'-type view for open issues.

# Coding Workflow


### Step 0) Choosing or creating an Issue for work to be done

Before starting work, we prefer to have an issue logged in Github. If you don't have an issue assigned
to yourself, the open issues for the current sprint should be in the "Ready" column of the "Boards"
tab. You can pick one up by assigning it to yourself and moving it to "In Progress". If there isn't
an issue for the work you have in mind, you can click "New Issue" in the "Boards" or "Issues" tabs.
A detailed description with accurate tags are much appreciated.

### Step 1) Hacking

#### &#10146; Clone the given Repository from GitHub

Example: `git clone http://github.com/mitodl/micromasters.git`

#### &#10146; Create a Branch for the given issue

Assuming you're working on issue **#123** in GitHub, you create a new branch from master/main like so:

    git checkout -b NEW-BRANCH_NAME origin/master

Eventually you'll want to push this new branch to origin to make it available to other developers.
You can do this before you open a Pull Request (see below) or right after you create the branch:

    git push -u origin NEW-BRANCH_NAME

#### &#10146; Hack away!

Make changes, test (automated and/or manual), commit, repeat. The repo README should have details
for running automated tests.


### Step 2) Code Review

_When you've completed the given issue, the code is passing tests, and you feel like it's ready
to ship, it's time to start code review. Make sure you read through the
[Code Review guide](https://github.com/mitodl/handbook/blob/master/code-review.md#everyone) to get a sense
about why we do code reviews and how to approach them, both as a reviewer and reviewee._

#### &#10146; Open a Pull Request ("PR")

This can be kicked off from several places in GitHub. The easiest one to remember is the "New
Pull Request" button in your repo's "Pull Requests" tab
([example](https://github.com/mitodl/micromasters/pulls)). Your **base** branch should be
**master/main** and your **compare** branch should be your issue branch.

When you first open the PR, the input should be pre-populated with a
PR template. Each project has its own template, but it should look roughly like [this](https://github.com/mitodl/handbook/blob/master/pr-template.md). Fill in every section
that is relevant to your PR, and delete the sections that aren't relevant. Under the **"relevant
tickets"** section, be sure to indicate the issue that you're fixing. GitHub will automatically
link to your issue if you type '#' followed by your issue number (and it will help you with an
autocomplete menu). **If your PR is intended to fix/close an issue (typically true), fill in this
section with "fixes #123" or "closes #123".** When your PR is eventually merged, this text is
parsed, and the issue will be automatically closed if "fixes #X"/"closes #X" is found.

Before (or after) you submit the PR, add the "Needs Review" label to it.


**TL;DR**

1. Click "New Pull Request"
1. Fill in (or confirm) the appropriate **base** and **compare** branches
1. Fill in all the relevant sections in the PR template (especially "relevant tickets", which should
   typically read "Closes #*ISSUE_NUMBER*" or "Fixes #*ISSUE_NUMBER*")
1. Add the "Needs Review" label
1. Submit and wait for comments/approval

#### &#10146; Respond to comments until you get approval

As comments come in and changes are requested, you can make any necessary changes and push them when
you feel the requests are satisfied.

Reviewers should change the PR's label to "Waiting on author" when they are finished making comments
and requesting changes. Similarly, when you are finished making changes based on that feedback and
responding to comments, you should change the label back to "Needs Review".

Right now, we consider a PR to be "approved" if the assigned reviewer on the PR has given you a +1
(thumbs up). Use your own discretion about seeking out further approvals. That might be appropriate
if there is someone very familiar with the code you're working on, but isn't the assigned reviewer.

In the unlikely event of a disagreement that can't be resolved, the final decision should be taken by
the developer and not the reviewer. Again, you should read the
[Code Review guide](https://github.com/mitodl/handbook/blob/master/code-review.md#everyone), which has
some advice on avoiding this kind of unresolvable disagreement.

**TL;DR**

- Make changes/respond to comments as needed and reset the PR label to "Needs Review"
- PR is considered "approved" when you get a +1 (thumbs up) from the assigned reviewer

#### &#10146; (As necessary) Rebase onto master/main and resolve conflicts when they occur

As you work on your PR, changes may be made to master/main that cause would-be merge conflicts. The
merge-ability of your branch will be indicated near the bottom of your PR page. If conflicts are detected,
you should rebase your branch onto master/main, fix the conflicts, and push your branch. Here is a [rebasing
guide maintained by edX](https://github.com/edx/edx-platform/wiki/How-to-Rebase-a-Pull-Request). The
process looks something like this in the command line:

    # Assuming (a) your issue branch is currently checked out...
    git fetch && git rebase origin/master
    # Resolve merge conflicts, commit, repeat until done
    # When merge conflicts are all resolved...
    git push

That message near the bottom of your PR page should have a green check mark and read "This branch has no
conflicts with the base branch" when you have successfully resolved and pushed.


### Step 3) Merging

#### &#10146; "Squash" the commits on your branch

You may have many incremental commits on your issue branch. In order to keep the git log for the master/main
branch clean and readable, we "squash" all issue branch commits into one commit before merging. The commit
message should be a past tense description of your entire PR, eg: "Updated the home page to match new design". Using
the past tense makes it easier to write coherent release notes.

The actual squashing can be done in a few different ways. The two main approaches are (1) interactive rebase,
and (2) soft reset. The aforementioned edX rebasing guide contains information about
[squashing branch commits with an interactive rebase](https://github.com/edx/edx-platform/wiki/How-to-Rebase-a-Pull-Request#squash-your-changes).
This is the recommended method, but you can also try accomplishing this with a soft reset:

    git fetch
    git merge-base YOUR_BRANCH master
    # 'merge-base' will give you a commit hash. Copy that value for this next command
    git reset --soft COMMIT_HASH
    git commit -am "YOUR_COMMIT_MESSAGE"
    # The above will cause your branch to diverge, so you'll need force-push.
    # Make sure your git 'push.default' config is set to 'simple' before running this.
    git push origin YOUR_BRANCH -f

When this is done, your git log should show one commit for all the changes in your branch followed by the
'merge base' of your branch.

#### &#10146; Merge your PR into master/main via GitHub

We merge PR's by button-click in GitHub for a few reasons. One of them to avoid possible human error in
pushing to master/main in the command line. Near the bottom of your PR page, you should see a "Merge pull request"
button. Click that.

#### &#10146; Delete your branch and move your issue

You should see an option to delete your branch after you merge in GitHub. When the branch is deleted, go to
the "Boards" tab in your repo and move your issue over to "Done".
