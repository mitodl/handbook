---
layout: default
title: Release Process
has_children: true
parent: How We Deliver
---

## Release Responsibilities

### Communication

Communication around releases is important:

- The main channel for communication around releases is the Slack channel for the product associated with the release.
- If you won't be available to verify your changes in the RC environment, ask a team member to take over and verify your changes in RC.
- If a release candidate should not be promoted to the Production environment, identify and communicate the next steps within the Slack channel for the product associated with the release.


### Engineering Team

Collectively the team members who have made changes in a release candidate are responsible for coordinating the release.

### Release Manager

When a new release candidate is created, the role of "release manager" is designated to the engineer who triggered it.


## When can releases happen?

### Release Candidates
  
Engineers create release candidates whenever they need to.  Release candidates (releases to the CI and QA environments) can be performed whenever, unless a hold or blocker as been communicated by another team member.  

- Consider the following when there is already a release candidate in progress:
  - If your changes **aren't** urgent, wait for this release candidate to be completed.
  - If your changes **are** urgent, communicate with the rest of the team before closing the current one and starting a new one. There may be other urgent work in the queue ahead of you that can't wait for a release candidate to be refreshed or it's undesirable to have other changes included.

### Production Releases

Production releases should only be done Monday through Thursday, 8am-4pm EST.  This mitigates the risk of regressions or unanticipated production bugs from occurring during off-hours.

Exceptions to this rule are up to the team's discretion, but some general cases where this might be done:

- Production is completely down
- Bug in the critical path of the application
- Delivery of certain functionality is urgent (typically this is due to deadline timing of other teams)