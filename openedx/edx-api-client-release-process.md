---
parent: OpenedX
---

# edx-api-client Release Process

This document describes the process for publishing a new release of the `edx-api-client` package to PyPI.

## Steps to Publish a New Release

1. **Go to Slack**
   - Open the `doof-edx-api-client` Slack channel.
   - Send the message: `@doof release notes`
   - This will cut a new release using the latest commits from the `master` branch.

2. **Test the Release**
   - Ensure you have tested the changes locally.
   - You can use the `release-candidate` branch to test the latest changes that will go to PyPI.
   - After thorough testing, check the box in the new release on GitHub to confirm testing is complete.

3. **Publish to PyPI**
   - Go back to the Slack channel.
   - Send the message: `@doof publish <new-version>`.
   - This will publish the new release to PyPI automatically.

---
