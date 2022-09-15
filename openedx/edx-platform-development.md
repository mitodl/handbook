---
parent: OpenedX
---
# Platform Development

### Purpose

At MIT ODL, we often work on edX platform features and bugfixes. There are [multiple named releases of Open edX](https://openedx.atlassian.net/wiki/spaces/DOC/pages/11108700/Open+edX+Releases) and a couple different methods of running the platform on your own machine. This guide aims to be the source of truth for ODL developers in terms of choosing the correct repo/branch for development, running devstack, and troubleshooting an installation.

### Current State of Things

*(updated 04/02/2018)*

This is the state of `edx-platform` development at MIT ODL. If this doesn't make sense, read below.

- MIT ODL main development branch: **`mitx/master`**
- Accepted method for running devstack on that branch: **Docker**
- Unless an issue explicitly says otherwise, you should always do `edx-platform` work in the MIT ODL fork, in a branch off of the main development branch specified above, and any PRs should be opened against that branch.

### Running devstack

There are 2 different methods for running devstack on your own machine: Vagrant ([guide](https://openedx.atlassian.net/wiki/spaces/OpenOPS/pages/60227787/Running+Vagrant-based+Devstack)) and Docker ([guide](https://github.com/edx/devstack)). The edX team is focusing all future development on the Docker-based method, starting after the Ginkgo release. MIT ODL developers should run devstack using these guidelines:

1. If you are working on a branch based on the `Ginkgo` release or any release before that, use **Vagrant**. A pre-built vagrant box based on the `mitx/ginkgo` fork can be downloaded [here](https://s3.amazonaws.com/public.mitx.mit.edu/vagrant/mitx-ginkgo/mitx_devstack_ginkgo.tar.gz).
2. If you are working on a branch based on any release that came after `Ginkgo` (`Hawthorn`, etc.), use **Docker**

### `edx-platform` Repository/Branch

MIT ODL maintains a fork of the `edx-platform` repo.

- MIT ODL fork: https://github.com/mitodl/edx-platform
- Upstream edX repo: https://github.com/edx/edx-platform

The MIT ODL fork is used for most/all MIT courses that run on the edX platform. Almost all of our edX development work centers around feature additions and bug fixes for MIT courses, so we do most of our work in that fork. We maintain our own IT infrastructure for MIT courses (separate from edX). The 'main development branch' specified above is the branch that releases are created from. We do to put up some PRs against the upstream edX-maintained repo, mostly to make it easier to keep our fork up to date with that repo.
