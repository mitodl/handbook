# Release Process

## Overview

Our release process prepares an application for
release to users. Releases are made in the context of a GitHub
repository. This document describes the steps required.

In addition to creating a code snapshot for production, the release process
triggers code deployments.  There is already tooling that deploys commits to
``master/main`` to the ``<projectname>-ci`` server. The release process triggers
deployments to ``<projectname>-rc`` for acceptance testing.  When
``release-candidate`` is merged to ``release``, the application is deployed
to the production server.

## Steps
### 1. Check-out a current ``master/main`` branch
Verify that ``master/main`` contains the commits intended for the release.  If
there isn't already an issue in GitHub for the release, create one.
### 2. Create a ``release-candidate`` branch and hard reset it to ``master/main``
The ``release-candidate`` branch is hard reset to ``master/main`` since for the
purposes of the release process, its state on the remote is irrelevant.
### 4. Generate release notes
All of our projects contain a file for release notes in the project
root directory.  This file is named ``RELEASE.rst``.  For each release it is
updated with a section titled "Release <version-number>" containing a list
of the commits in the release.  
### 5. Update version numbers
Project version numbers are typically stored in ``settings.py`` and/or
``setup.py`` depending on the application's architecture and purpose.
### 6. Commit updates and push ``release-candidate`` branch
### 7. Generate release notes with checkboxes
### 8. Open PR to merge ``release-candidate`` branch into ``release`` branch
### 9. Merge PR once developers verify their commits (manual step)
### 10. Tag the release
### 11. Merge the ``release`` branch into the ``master/main`` branch and push
### 12. Send email notifications (manual step)


## Release Version Numbers
Our releases are identified by a version number that follows the
[Semantic Versioning scheme]([http://semver.org/).

## Release Automation Script
The release process is partially automated through a BASH script.  The script
is available in a public GitHub repository: [mitodl/release-script](https://github.com/mitodl/release-script)

## Version Tags
Version tags are composed by prefacing the release version number with the
letter ``v``.  For example, the tag for release version ``0.3.0`` is
``v0.3.0``.

Version tags are applied to the ``release`` branch after the
``release-candidate`` branch is merged.
