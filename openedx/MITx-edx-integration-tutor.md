---
parent: OpenedX
---
# Integrating MIT Applications with Open edX using Tutor

| Application | PORT | Domain               |
|-------------|------|----------------------|
| MITxPro     | 8053 | xpro.odl.local       |
| MITxOnline  | 8013 | mitxonline.odl.local |

In order to create user accounts in Open edX and permit authentication from MIT Application to Open edX, you need to configure MIT Application as an OAuth2 provider for Open edX.

    These instructions should work for a Tutor Dev or Tutor Nightly deployment as well. Specify `--tutor-dev` instead of `--tutor` when running `configure_instance` so the URLs have a port on them.

## Preliminary Step

`pyenv` (and `pyenv-virtualenv`) are highly recommended for managing local Python versions. Use the instructions on their [GitHub page](https://github.com/pyenv/pyenv) to install if you haven't already installed it.

You'll want to create at least a virtualenv for Tutor. As of this writing, Tutor uses Python 3.8.12 (in the LMS container at least); I have also successfully used 3.9.16. 3.11 has *not* worked for me.

## Tutor Setup, Part One

    Note that no hosts file changes are needed if you use the default `local.openedx.io` domain - that's a real domain with a wildcard subdomain cname that points to 127.0.0.1.

To begin, you need to follow the [Installing Tutor for development](https://docs.tutor.edly.io/dev.html#open-edx-development) instructions provided by Tutor **for local development installations**. Do this with your Tutor virtualenv activated.

Once Tutor has bootstrapped itself and is available, create a superuser account:

    tutor dev do createuser --staff --superuser edx edx@example.org

Supply a password (the one used by devstack is `edx` so use that if you want to be consistent with it).

Create a service worker for MITx Application

    tutor dev do createuser --staff mit_Application_serviceworker service@mitx.odl.local

Supply a password (this one doesn't matter for a local deployment, you won't ever actually use the account).

### For MITxOnline Only
For best results, create two new courses within edX. The MITxOnline `configure_instance` command expects a couple of courses to exist in edX (because they come with the devstack package):

| Course ID                       | Course Title         |
|---------------------------------|----------------------|
| course-v1:edX+DemoX+Demo_Course | Demonstration Course |
| course-v1:edX+E2E-101+course    | E2E Test Course      |

If you have a devstack instance handy, you can export these and import them into Tutor. Otherwise, just create them and make sure to set dates for the courses (they default to 2030 otherwise).

### Configure Open edX user and token for use with MITx Application management commands

- In Open edX, under `http://local.openedx.io:8000/admin/oauth2_provider/accesstoken/` add access token with that newly created staff user.

### MIT Application Setup

To set up MIT Application:

1. Get the gateway IP for the `EDX_APP`
   1. Linux users: The gateway IP of the docker-compose networking setup for edx LMS `docker network inspect tutor_dev_default | grep Gateway`.
   2. OSX users: Use `host.docker.internal`

2. Set up your `.env` file. These settings need particular attention:

   - `OPENEDX_IP`: set to the gateway IP from the first step.
   - `OPENEDX_API_BASE_URL`: set to `http://<EDX_HOSTNAME>:<PORT>`
   - `OPENEDX_SERVICE_WORKER_USERNAME`: set to `mit_Application_serviceworker` (unless you changed this)
   - `OPENEDX_SERVICE_WORKER_API_TOKEN`: set to the token you just generated
   - `OPENEDX_OAUTH_PROVIDER`: set to `ol-oauth2`
   - `OPENEDX_SOCIAL_LOGIN_PATH`: set to `/auth/login/ol-oauth2/?auth_entry=login`

### Run the configure_instance command

    docker-compose run --rm web ./manage.py configure_instance linux --gateway <ip> --tutor-dev

  Where `<ip>` is the IP from the first step.(On macOS, specify macos instead of linux. You can also skip --gateway.) You will need to supply passwords for the MITx Application superuser and test learner accounts. **Make a note of the client ID and secret that it will print out at the end.**


## Tutor Setup, Part Two

Note that some of these steps require editing the main configuration files for the production instance (which is also used for a local deployment). Most of the settings that need to be adjusted to get integration working are overridden by the default Tutor configuration, so you can't update them by setting `config.yml`. If you're using the development Tutor build, you'll likely need to edit `development.py` rather than `production.py` as necessary.

These steps will also disable the AuthN SSO MFE, so from here on you'll get normal edX authentication screens (if you're not being bounced to MITx Application).


1. Get the gateway IP of the `mitxApplication_default` Docker network. Example:

       docker network inspect mitxpro_default | grep Gateway

2. Log into to edX using your superuser account, and make sure your session stays open. Sessions are pretty long-lived so this just means not closing the browser that you started the session in. (Part of this process will involve mostly breaking authentication so it's important that you are able to get into the admin.)
3. Stop Tutor: `tutor dev stop`
4. Add extra requirements required for OAuth Configuration:

       tutor config save --append OPENEDX_EXTRA_PIP_REQUIREMENTS=ol-social-auth
       tutor config save --append OPENEDX_EXTRA_PIP_REQUIREMENTS=openedx-companion-auth

   **NOTE**: Reference for [Installing extra xblocks and requirements](https://docs.tutor.edly.io/configuration.html#installing-extra-xblocks-and-requirements)

5. **If you have cloned and mounted `edx-platform` while setting up the Tutor**, do this step and skip steps 6, 7, and 8. You can create `private.py` file at `edx-platform/lms/envs/{here}` and add the following configurations to allow additional OAuth providers

   ```
   from .production import AUTHENTICATION_BACKENDS, FEATURES, IDA_LOGOUT_URI_LIST, REGISTRATION_EXTRA_FIELDS

   FEATURES["ALLOW_PUBLIC_ACCOUNT_CREATION"] = True
   FEATURES["SKIP_EMAIL_VALIDATION"] = True

   REGISTRATION_EXTRA_FIELDS["country"] = "hidden"

   THIRD_PARTY_AUTH_BACKENDS = ["social_auth_mitxpro.backends.MITxProOAuth2",]

   AUTHENTICATION_BACKENDS = list(THIRD_PARTY_AUTH_BACKENDS) + list(AUTHENTICATION_BACKENDS)

   IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://{Domain}:{PORT}/logout"])

   SOCIAL_AUTH_OAUTH_SECRETS = {
      "ol-oauth2": <mit_app_client_secret>  // you just copied from configure_instance command output
   }
   ```

6. Change into the configuration root for Tutor:

        cd "$(tutor config printroot)"

7. Edit the `env/apps/openedx/config/lms.env.yml` file and add:

        FEATURES:
            ALLOW_PUBLIC_ACCOUNT_CREATION: true
            SKIP_EMAIL_VALIDATION: true

   The `FEATURES` block (should be at the top).

8. Edit the `env/apps/openedx/settings/lms/development.py` settings file.

   - Add to the end of the file:

      - `THIRD_PARTY_AUTH_BACKENDS = ['social_auth_mitxpro.backends.MITxProOAuth2']`
      - `REGISTRATION_EXTRA_FIELDS["country"] = "hidden"`
      - `AUTHENTICATION_BACKENDS.append('social_auth_mitxpro.backends.MITxProOAuth2')`
      - `SOCIAL_AUTH_OAUTH_SECRETS = {"ol-oauth2": <MIT_Client_Secret> }` - Client Secret that you just copied after `configure_instance` management command
      - `IDA_LOGOUT_URI_LIST.append('http://{Domain}:{PORT}/logout/')`

    - Find and update:

      - `FEATURES['ENABLE_AUTHN_MICROFRONTEND'] = False` (defaults to True)

9. Build a new `openedx` image: `tutor images build openedx-dev` (this will take a long time)
10. Run a Docker Compse rebuild: `tutor dev dc build` (this should be pretty quick - it's likely not required, just doing it here for safety)
11. Restart Tutor: `tutor dev start -d` (omit `-d` if you want to watch the logs)
12. Check your settings. There's a `print_setting` command that you can use to verify everything is set properly:

    - `tutor dev run lms ./manage.py lms print_setting REGISTRATION_EXTRA_FIELDS`
    - `tutor dev run lms ./manage.py lms print_setting AUTHENTICATION_BACKENDS`
    - `tutor dev run lms ./manage.py lms print_setting FEATURES` - will print a lot of stuff
    - `tutor dev run lms ./manage.py lms print_setting THIRD_PARTY_AUTH_BACKENDS`
    - If you do have weird errors or settings not showing properly, make sure you edited the right yaml files *and* that they're using the right whitespace (i.e. don't use tabs).

13. In a separate browser session of some kind (incognito/private browsing/other browser entirely), try to navigate to `http://local.openedx.io:8000`. It should load but it should give you an error message. In the LMS logs, you should see an error message for "Can't fetch settings for disabled provider." This is proper operation - the OAuth2 settings aren't in place yet.
14. In the superuser session you have open, go to `http://local.openedx.io:8000/admin`. This should work. If you've been logged out, you should still be able to get in. If you can't (for instance, if you're getting 500 errors), you will need to turn off `ENABLE_THIRD_PARTY_AUTH` in `FEATURES`, restart Tutor using `tutor dev stop` and `start`, not using `reboot`, then try again.
15. Go to `http://local.openedx.io:8000/admin/third_party_auth/oauth2providerconfig/add/` and add a provider configuration:

    - Enabled is **checked**.
    - Name: `Login with MIT App`
    - Slug: `ol-oauth2`
    - Site: `local.openedx.io:8000`
    - Skip hinted login dialog is **checked**.
    - Skip registration form is **checked**.
    - Skip email verification is **checked**.
    - Sync learner profile data is **checked**.
    - Enable sso id verification is **checked**.
    - Backend name: `ol-oauth2`
    - `Client ID` and `Client Secret`: from record created by `configure_instance` when you set up MITx Application.
    - Other settings:

          {
             "AUTHORIZATION_URL": "http://{Domain}:{PORT}/oauth2/authorize/",
             "ACCESS_TOKEN_URL": "http://<MITxApplication_GATEWAY_IP>:<PORT>/oauth2/token/",
             "API_ROOT": "http://<MITxApplication_GATEWAY_IP>:<PORT>/"
          }

     where MITxApplication_GATEWAY_IP is the IP from the `mitxApplication_default` network from the first step. **Mac users**, use `host.docker.internal` for MITxApplication_GATEWAY_IP.

### Configure Open edX to support OAuth2 authentication from MITx Application

   - Go to `http://local.openedx.io:8000/admin/oauth2_provider/application/` and add/edit the `edx-oauth-app` entry.
   - Ensure these settings are set:

      - Name: `edx-oauth-app`
      - Redirect uris: `http://{Domain}:{PORT}/login/_private/complete`
      - Client type: `Confidential`
      - Authorization grant type: `Authorization code`
      - Skip authorization is checked.

   - Save `Client id` and `Client secret`.

Update your MIT Application `.env` file. Set `OPENEDX_API_CLIENT_ID` and `OPENEDX_API_CLIENT_SECRET` to the values from the record you created or updated in the last step.

Also set the **LOGOUT_REDIRECT_URL** in `.env`:

    LOGOUT_REDIRECT_URL=http://local.openedx.io:8000/logout

  - Build the MIT Application: `docker-compose build`

You should now be able to run some MIT Application management commands to ensure the service worker is set up properly:

   - `sync_courserun --all ALL` should sync the two test courses (if you made them).
      - `sync_courseruns --all ALL` in MITxPro
   - `repair_missing_courseware_records` should also work.

In the separate browser session, attempt to log in again. This time, you should be able to log in through MIT Application, and you should be able to get to the edX LMS dashboard. If not, then double-check your provider configuration settings and try again.

   - If you are still getting "Can't fetch settings" errors, **make sure** your Site is set properly - there are three options by default and only one works. (This was typically the problem I had.)

**Optionally**, log into the LMS Django Admin and make your MIT Application superuser account a superuser there too.

## Other Notes

**Trying to set configuration settings via `tutor config` will undo the specialty configuration above.** If you need to make changes to the configuration, either manually edit the `env/apps/openedx/config/lms.env.yml` file or the `env/apps/openedx/settings/lms/production.py` file and restart your Tutor instance.

**Make sure your service worker account is active.** It's an easy checkbox to miss.

**Restarting** If you want to rebuild from scratch, make sure you `docker image prune`. It's also recommended to remove the Tutor project root folder - `tutor config printroot` will tell you where that is.

**Running Multiple Tutor Instances** If you want to run more than one Tutor instance, it's pretty important to specify the project root explicitly or you may end up with one instance trying to use config files from another and things getting confused from there. [`See the Tutor documentation for this.`](https://docs.tutor.overhang.io/local.html#tutor-root) (A suggestion: configure aliases to the `tutor` command that run `tutor --root=<whatever>` so you don't have to rely on environment variables, especially if you keep multiple terminal sessions going.)
