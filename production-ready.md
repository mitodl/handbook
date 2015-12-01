# Production ready

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

## Production checklist

- [ ] Does it support configuration via environment variables?
- [ ] Does it support configuration via flatfile (e.g. YAML)?
- [ ] Does it have a sane default configuration (e.g. as close to a realistic production configuration as possible)?
- [ ] Can it run safely in parallel (e.g. on many uWSGI or Gunicorn HTTP workers)?
- [ ] Can it log to standard in/out?
- [ ] Can it log to syslog?
- [ ] Can it send stack trace emails to a configurable email list?
- [ ] Does it have configurable log levels (e.g. DEBUG, ERROR, WARNING, INFO)?

## Release process
