---
parent: OpenedX
---

# edx-api-client Release Process

This document describes the process for publishing a new release of the `edx-api-client` package to PyPI.

## Steps to Publish a New Release

1. **Go to Slack**
   - Open the `doof-edx-api-client` Slack channel.
   - Cut a new release using `@doof release notes` and then selecting the appropriate version.
   - This will cut a new release and create a PR using the latest commits from the `master` branch.

2. **Test the Release**
   - Test the release PR locally, and check all the boxes. (Never merge this PR manually)

3. **Publish to PyPI**
   - Go back to the `doof-edx-api-client` Slack channel.
   - Publish the new release to PyPI using `@doof publish <new-version>` (new-version is the version that you selected upon cutting a new release).

---
