---
layout: default
title: Release Process
has_children: true
parent: How We Deliver
---

Some of these responsibilities may differ depending on the type of project, please refer to those documents for more specifics.

## Responsibilities

This document outlines high-level responsibilities

### Communication

Communication around releases is important:

- The main channel for communication around releases is the Slack channel for the product in question.
- Engineers are free to augment Slack with other tools (e.g. Zoom) as they see fit.
- If you won't be available to verify your changes in RC, communicate this to the team and ask a team member to do this in your absence before merging changes.
- If a release candidate isn't viable for release, communicate this and what the next step(s) are.

### Individual Engineer

As an engineer you have responsibilities to ensure the work you've done is reliably delivered.

After your code is merged:

- You may wait for the daily automated release candidate.
- If you have a more urgent need to get your changes released, you may trigger a manual release candidate. You should coordinate with other engineers who have also merged changes to ensure they are available to verify their own work, if applicable.

### Engineering Team

Collectively the team members who have made changes in a release candidate are responsible for coordinating the release.

### Release Manager

When a new release candidate is created, a release manager is designated automatically based on the following logic:
  
- For **manually triggered** release candidates, the engineer who triggered it.
- For **automated daily releases**, an engineer is selected at random from those who made commits. Preference is given to engineers working in the EST timezone so that we maximize overlap between their working hours and the production release hours.


## When can releases happen?

### Release Candidates
  
Engineers are welcome to cut release candidates whenever they need to, even during EST off-hours.

### Production Releases

Production releases should only be done Monday through Thursday, 8am-4pm EST to mitigate regressions or unanticipated production bugs from occurring off-hours.

Exceptions to this rule are up to the team's discretion, but some general cases where this might be done:

- Production is completely down
- Bug in the critical path of the application
- Delivery of certain functionality is urgent (typically this is due to deadline timing of other teams)

## Creating Release Candidates

- On a daily basis each project has a release candidate created automatically if there are new changes.
- As an engineer you may want a release candidate sooner than the next scheduled one:
  - You want additional time to be able to test your changes without holding up other work.
  - You have work that needs to be in production sooner.
- If there is already a release candidate in progress:
  - If your changes **aren't** urgent, wait for this release candidate to be completed.
  - If your changes **are** urgent, communicate with the rest of the team before closing the current one and starting a new one. There may be other urgent work in the queue ahead of you that can't wait for a release candidate to be refreshed or it's undesirable to have other changes included.