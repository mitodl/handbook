# Managing Heroku configuration variables

## Which Heroku configuration variables are managed by salt-proxy?
The following Heroku application configuration variables are managed by salt-proxy:
- [Bootcamp](https://github.com/mitodl/bootcamp-ecommerce)
- [Open](https://github.com/mitodl/open-discussions)
- [xPRO](https://github.com/mitodl/mitxpro)

## What pillar file needs to be modified?
In order for changes you make to `app.json` to be reflected in a Heroku application configuration variables, the keys/values need to be added to the app's respective pillar file below:
- [Bootcamp pillar](https://github.com/mitodl/salt-ops/blob/master/pillar/heroku/bootcamps.sls)
- [Open pillar](https://github.com/mitodl/salt-ops/blob/master/pillar/heroku/discussions.sls)
- [xPRO pillar](https://github.com/mitodl/salt-ops/blob/master/pillar/heroku/xpro.sls)

## What is the process to add/modify a Heroku configuration variable?
1. Ensure that the Heroku app you are interested in is managed by salt-proxy
2. If the value for the key being added/modified is the same for all envs (CI/RC/Production), then just add the key/value under heroku:config_vars. If the values differ for each env, then add the key/value under each env and reference it under heorku:config_vars
3. Submit a PR so that someone in DevOps reviews it
4. Once PR is merged, the change takes place the next time the salt-proxy's scheduled tasks runs. Currently, that happens every 5 days. If that's too long, please add a note in the PR and someone from DevOps can trigger the salt state to run.

## What about secret values?
We use vault to manage secrets and if one of the values modified should be managed by vault, simply follow the vault paths in the existing pillar files and if you need some additional clarification, please don't hesitate to ask someone from DevOps.