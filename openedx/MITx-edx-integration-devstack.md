---
parent: OpenedX
---
# Integrating MIT Applications with Open edX using Devstack

| Application | PORT | Domain               |
|-------------|------|----------------------|
| MITxPro     | 8053 | xpro.odl.local       |
| MITxOnline  | 8013 | mitxonline.odl.local |

In order to create user accounts in Open edX and enable authentication from MIT Application to Open edX, you need to configure MIT Application as an OAuth2 provider for Open edX.

## Setup Open edX Devstack

Following steps are inspired by [edx-devstack](https://github.com/edx/devstack).

## Add `/etc/hosts` alias for Open edX

If one doesn't already exist, add an alias to `/etc/hosts` for Open edX. We have standardized this alias
to `edx.odl.local`. Your `/etc/hosts` entry should look like this::

`127.0.0.1  edx.odl.local`

## Clone edx/devstack

    git clone https://github.com/edx/devstack
    cd devstack
    make requirements
    make dev.clone

## Pull latest images and run provision

    make pull
    make dev.provision

## Start your servers

    make dev.up

## Stop your servers

    make stop

## Setup social auth

### Install `social-auth-mitxpro` in LMS

There are two options for this:

#### Install via pip

    pip install social-auth-mitxpro

#### Install from local Build

- Checkout the [social-auth-mitxpro](https://github.com/mitodl/social-auth-mitxpro) project and build the package per the project instructions
- Copy the `social-auth-mitxpro-$VERSION.tar.gz` file into devstack's `edx-platform` directory
- In devstack, run `make lms-shell` and within that shell `pip install social-auth-mitxpro-$VERSION.tar.gz`

    - To update to a new development version without having to actually bump the package version, simply `pip uninstall social-auth-mitxpro`, then install again

### Install `openedx-companion-auth` in LMS

There are two options for this:

#### Install via pip

    pip install openedx-companion-auth

#### Install from local Build

- Checkout the [openedx-companion-auth](https://github.com/mitodl/open-edx-plugins/tree/main/src/openedx_companion_auth) project and build the package per the project instructions
- Copy the `openedx-companion-auth-$VERSION.tar.gz` file from the `dist` folder into devstack's `edx-platform` directory
- In devstack, run `make lms-shell` and within that shell `pip install openedx-companion-auth-$VERSION.tar.gz`

  - To update to a new development version without having to actually bump the package version, simply `pip uninstall -y openedx-companion-auth`, then install again
---------------------------------------------------------------------------------------------------------------------------------------------------
## Configure Open edX user and token for use with Application management commands

- In Open edX, create a staff user `mit_application_serviceworker` and then under `/admin/oauth2_provider/accesstoken/` add access token with that newly created staff user.

## MIT Application Setup

To set up MIT Application:

1. Get the gateway IP for the `EDX_APP`
   1. Linux users: You can use `docker network inspect <LMS_COTNAINER_NAME> | grep Gateway`
   2. OSX users: Use `host.docker.internal`

2. Set up your `.env` file. These settings need particular attention:

   - `OPENEDX_IP`: set to the gateway IP from the first step.
   - `OPENEDX_API_BASE_URL`: set to `http://<EDX_HOSTNAME>:<PORT>`
   - `OPENEDX_SERVICE_WORKER_USERNAME`: set to `mit_Application_serviceworker` (unless you changed this)
   - `OPENEDX_SERVICE_WORKER_API_TOKEN`: set to the token you just generated

   - Build the MIT Application: `docker-compose build`

## Run the `configure_instance` management command

    docker-compose run --rm web ./manage.py configure_instance linux --gateway <ip>

  Where `<ip>` is the IP from the first step. (On macOS, specify macos instead of linux. You can also skip --gateway.) You will need to supply passwords for the MIT Application superuser and test learner accounts. **Make a note of the client ID and secret that it will print out at the end.**

## Configure MIT application as a OAuth provider for Open edX

In Open edX (derived from instructions `[here](https://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/configuration/tpa/tpa_integrate_open/tpa_oauth.html#additional-oauth2-providers-advanced)`):

- Create `private.py` file at `edx-platform/lms/envs/{here}` and add the following configurations to allow additional OAuth providers

  ```
  from .production import AUTHENTICATION_BACKENDS, FEATURES, IDA_LOGOUT_URI_LIST, REGISTRATION_EXTRA_FIELDS

  FEATURES["ALLOW_PUBLIC_ACCOUNT_CREATION"] = True
  FEATURES["ENABLE_COMBINED_LOGIN_REGISTRATION"] = True
  FEATURES["ENABLE_THIRD_PARTY_AUTH"] = True
  FEATURES["ENABLE_OAUTH2_PROVIDER"] = True
  FEATURES["SKIP_EMAIL_VALIDATION"] = True

  REGISTRATION_EXTRA_FIELDS["country"] = "hidden"

  THIRD_PARTY_AUTH_BACKENDS = ["social_auth_mitxpro.backends.MITxProOAuth2",]

  AUTHENTICATION_BACKENDS = list(THIRD_PARTY_AUTH_BACKENDS) + list(AUTHENTICATION_BACKENDS)

  IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://{Domain}:{PORT}/logout"])

  SOCIAL_AUTH_OAUTH_SECRETS = {
    "mitxpro-oauth2": <mit_app_client_secret>  // you just copied from configure_instance command output
  }
  ```

- Login to django-admin (default username and password can be found `[here](https://github.com/openedx/devstack#usernames-and-passwords)`), go to `http://edx.odl.local:18000/admin/third_party_auth/oauth2providerconfig/`, and create a new config:

  - Select the default example site
  - The slug field **MUST** match the the backend's name, which for us is `mitxpro-oauth2`
  - Check the following checkboxes:

    - Enabled
    - Skip hinted login dialog
    - Skip registration form
    - Sync learner profile data
    - Enable SSO id verification
  - Set Backend name to: `mitxpro-oauth2`

  - In "Other settings", put:

    ```
    {
        "AUTHORIZATION_URL": "http://{Domain}:{PORT}/oauth2/authorize/",
        "ACCESS_TOKEN_URL": "http://<EXTERNAL_MIT_APP_HOST>:<PORT>/oauth2/token/",
        "API_ROOT": "http://<EXTERNAL_MIT_APP_HOST>:<PORT>/"
    }
    ```

  - `EXTERNAL_MIT_APP_HOST` will depend on your OS, but it needs to be resolvable within the edx container

    - Linux users: The gateway IP of the docker-compose networking setup for MIT Application as found via `docker network inspect mit-Application_default | grep Gateway` e.g; `docker network inspect mitxonline_default | grep Gateway`
    - OSX users: Use `host.docker.internal`

  - Save the configuration.

## Configure Open edX to support OAuth2 authentication from MITx Application

   - Go to `http://edx.odl.local:18000/admin/oauth2_provider/application/` and add/edit the `edx-oauth-app` entry.
   - Ensure these settings are set:

      - Name: `edx-oauth-app`
      - Redirect uris: `http://{Domain}:{PORT}/login/_private/complete`
      - Client type: `Confidential`
      - Authorization grant type: `Authorization code`
      - Skip authorization is checked.

   - Save `Client id` and `Client secret`.

Update your MIT Application `.env` file. Set `OPENEDX_API_CLIENT_ID` and `OPENEDX_API_CLIENT_SECRET` to the values from the record you created or updated in the last step.

Also set the **LOGOUT_REDIRECT_URL** in `.env`:

    LOGOUT_REDIRECT_URL=http://edx.odl.local:18000/logout

  - Build the MIT Application: `docker-compose build`

You should now be able to run some MIT Application management commands to ensure the service worker is set up properly:

   - `sync_courserun --all ALL` should sync the two test courses (if you made them).
      - `sync_courseruns --all ALL` in MITxPro
   - `repair_missing_courseware_records` should also work.

In the separate browser session, attempt to log in again. This time, you should be able to log in through MIT Application, and you should be able to get to the edX LMS dashboard. If not, then double-check your provider configuration settings and try again.

   - If you are still getting "Can't fetch settings" errors, **make sure** your Site is set properly - there are three options by default and only one works. (This was typically the problem I had.)

**Optionally**, log into the LMS Django Admin and make your MIT Application superuser account a superuser there too.
