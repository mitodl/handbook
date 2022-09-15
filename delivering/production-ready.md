---
parent: How We Deliver
---
# Production Ready

This document defines our standard for production ready applications and
changes from an *operations* standpoint. The point at which the *product* is
ready can vary highly from project to project.

## Overview

### Applications

At the application scope, we adhere closely to Heroku's
[Twelve-Factor App](http://12factor.net/) guidelines as a baseline for
production readiness. The "Production checklist" lists out specific
implementation guidelines for making applications production ready.

### New code changes

Any time a new changeset is found to impede production readiness (for example,
a changeset might introduce functionality that, by design, obscures logging or
fails silently in a production setting), it's not a production ready change,
and should not be promoted in the release pipeline. Additionally, for any code
change to become "production ready", it must be verified working in the context
of a controlled pre-production environment identical to production.

## "Production readiness" checklists

### Required features:

- [ ] Does it support configuration via environment variables?
- [ ] Does it support configuration via flatfile (e.g. YAML)?
- [ ] Does it have a sane default configuration (e.g. as close to a realistic production configuration as possible)?
- [ ] Does it run on parallel HTTP workers? (e.g. uWSGI, Gunicorn)?
- [ ] Does it log to standard out/error?
- [ ] Does it log to syslog?
- [ ] Can it send stack trace emails to a configurable email list?
- [ ] Does it have configurable log levels (e.g. DEBUG, ERROR, WARNING, INFO)?
- [ ] Does it have a test suite which measures code coverage?
- [ ] Does it have a status route (which requires a secret token) that can be used for health checking by a monitoring service?
- [ ] Does the health check validate that each service the app depends on is reachable (e.g. databases, caching, external APIs when necessary)?
- [ ] Does the health check return a response body with the availability information above in a machine-readable format in all cases?
- [ ] Does the health check return a 500 status code when any component in the health check is detected to be failing?

### Required documentation/service infrastructure:

- [ ] Does it have continuous integration builds which run the full test suite?
- [ ] Does it have an automated continuous delivery to a development environment?
- [ ] Does it have a documented release process?
- [ ] Does its release process include staging and production builds, where staging builds are promoted to production?
- [ ] Does its release process require that code changes be "signed off" as working on staging before being promoted to production?
- [ ] Does it have application/resource monitoring in production (e.g. New Relic agent, Zenoss SNMP)?
- [ ] Does it have service availability monitoring in production (e.g. periodic health checks with Zenoss or New Relic)?
- [ ] Does it have automated, periodically generated, fully restorable backups for each non-ephemeral data store in production?
