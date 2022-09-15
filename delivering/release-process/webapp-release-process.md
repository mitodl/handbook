---
parent: Release Process
grand_parent: How We Deliver
title: Web Applications
---
# Web Application Release Process

## Responsibilities

### Individual Engineer

As an engineer you have responsibilities to ensure the work you've done is reliably delivered to production:


- Before your code is merged:
  - Coordinate with DevOps on having configuration in place both on RC and production for your changes _before_ it gets merged so that releases are not held up.
- After your code is merged:
  - When the changes have been deployed on RC, verify your changes with a round of functional testing.
- After your code is released:
  - Perform smoke testing of the functionality if it is prudent and won't disrupt other users.

### Engineering Team

Collectively the team members who have made changes are responsible for coordinating the release to production:

- The team members who have changes in the release candidate are responsible for verifying their changes:
  - The release candidate PR will contain a checklist of the included changes by commit message.
  - Each engineer should:
    - Perform a functional test of their changes
    - Check off their boxes as an indicator that they have done so and their changes are good to go to production.
    - If there is a bug or regression discovered, that should be communicated
- When all changes in the release candidate have been verified by the team, the engineer who has been designated Release Manager releases it to production.
  - The Release Manager should periodically check back to ensure the changes went out.
  - If the release failed, the Release Manager should investigate, communicating with other team members who are more informed about the problem area.

## Release Lifecycle

```mermaid
sequenceDiagram
  actor eng as Software Engineers
  participant branchmain as main/master <br> branch
  participant envci as CI <br> Environment

  participant branchrc as release-candidate <br> branch
  participant envrc as Release Candidate <br> Environment

  participant branchrelease as release <br> branch
  participant envprod as Production <br> Environment

  loop
    eng->>branchmain: Merge pull requests
  end

  critical Automated CI Deployments
    branchmain->>envci: Deploy to CI
  end

  Note over eng: Requests a release candidate

  critical New Release Candidate
    branchmain->>branchrc: Latest code merged
    branchrc->>branchrc: Release notes and PR generated
    branchrc->>envrc: Deploy to RC
    eng->>envrc: Engineer verifies their commit(s)
    note over eng: Checks off boxes on the PR
  end

  critical Production Release
    eng->>branchrc: Engineer coordinated release approval
    branchrc->>branchmain: Merged back to main
    branchmain->>branchrelease: Main branch merged to release
    branchrelease->>envprod: Deploy to Production
  end
```


## Versioning

Our releases are versioned using a date-based versioning scheme following the pattern `YYYY.MM.DD.#`:

- `YYYY` - full year
- `MM` - zero-padded month number
- `DD` - zero-padded day of the month
- `#` - release number starting at `0` for the first release that day and incrementing on each subsequent release

For example: 

- `2022.03.15.0` would be the first release on March 15, 2022.
- `2022.03.15.7` would be the seventh release on March 15, 2022.
- `2022.03.16.0` would be the first release on March 16, 2022 (release number resets the next day).


