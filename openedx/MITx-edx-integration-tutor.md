---
parent: OpenedX
---
# Integrating MIT Applications with Open edX using Tutor

| Application | PORT | Domain               |
|-------------|------|----------------------|
| MITxPro     | 8053 | xpro.odl.local       |
| MITxOnline  | 8013 | mitxonline.odl.local |

In order to create user accounts in Open edX and permit authentication from MIT Application to Open edX, you need to configure MIT Application as an OAuth2 provider for Open edX.

## Pre-requiste

To begin, you need to follow the [Installing Tutor for development](https://docs.tutor.edly.io/dev.html#open-edx-development) instructions provided by Tutor **for local development installations**. 

Once Tutor has bootstrapped itself and is available, create a superuser account:

    tutor dev do createuser --staff --superuser edx edx@example.org

Create a service worker for the MIT Application.

    tutor dev do createuser --staff mit_Application_serviceworker service@mitx.odl.local

Log in to your edX and MIT application as an admin and make sure the session remains active throughout the process. (Part of this process will involve mostly breaking authentication, so it's important that you are able to access the admin.)

#### For MITxOnline Only
For best results, create two new courses within edX. The MITxOnline `configure_instance` command expects a couple of courses to exist in edX (because they come with the devstack package):

| Course ID                       | Course Title         |
|---------------------------------|----------------------|
| course-v1:edX+DemoX+Demo_Course | Demonstration Course |
| course-v1:edX+E2E-101+course    | E2E Test Course      |

If you have a devstack instance handy, you can export these and import them into Tutor. Otherwise, just create them and make sure to set dates for the courses (they default to 2030 otherwise).

## Configure Open edX to support OAuth2 authentication from MIT Application

   - Go to [`http://local.openedx.io:8000/admin/oauth2_provider/application/`](http://local.openedx.io:8000/admin/oauth2_provider/application/) and add the `{app_name}-oauth-app` entry.
   - Ensure these settings are set:

      - Name: `{app_name}-oauth-app` example: `xpro-oauth-app`
      - Redirect uris: `http://{Domain}:{PORT}/login/_private/complete`
      - Client type: `Confidential`
      - Authorization grant type: `Authorization code`
      - Skip authorization is checked.

   - Save `Client id` and `Client secret`.

## Create an access token to use with MIT Application management commands

- In Open edX, under [`http://local.openedx.io:8000/admin/oauth2_provider/accesstoken/`](http://local.openedx.io:8000/admin/oauth2_provider/accesstoken/), create an access token with that newly created staff user. Select the `{app_name}-oauth-app` application you created in the previous step.

## MIT Application Setup

To set up MIT Application:

1. Get the gateway IP for the `EDX_APP`
   - Linux users: The gateway IP of the docker-compose networking setup for edx LMS

         docker network inspect tutor_dev_default | grep Gateway

   - OSX users: Use `host.docker.internal`

2. Set up your `.env` file. These settings need particular attention:

   - `OPENEDX_IP`: set to the gateway IP from the first step.
   - `OPENEDX_API_BASE_URL`: set to `http://local.openedx.io:8000`
   - `OPENEDX_SERVICE_WORKER_USERNAME`: set to `mit_Application_serviceworker` (unless you changed this)
   - `OPENEDX_SERVICE_WORKER_API_TOKEN`: set to the token you just generated
   - `OPENEDX_OAUTH_PROVIDER`: set to `ol-oauth2`
   - `OPENEDX_SOCIAL_LOGIN_PATH`: set to `/auth/login/ol-oauth2/?auth_entry=login`
   - `OPENEDX_API_CLIENT_ID`: set to the client id of the oauth application you created in the above steps
   - `OPENEDX_API_CLIENT_SECRET`: set to the client secret of the oauth application you created in the above steps
   - `LOGOUT_REDIRECT_URL`: `http://local.openedx.io:8000/logout`

    Run the `docker-compose up -d` command after setting these values

3. Run the configure_instance command

       docker-compose run --rm web ./manage.py configure_instance linux --gateway <ip> --tutor-dev

      Where `<ip>` is the IP from the first step. (On macOS, specify macos instead of linux. You can also skip --gateway.) You will need to supply passwords for the MIT Application superuser and test learner accounts. **Make a note of the client ID and secret that will print out at the end.**


## EdX Application Setup

1. Linux Users: Get the gateway IP of the `mitApplication_default` Docker network. Example:

       docker network inspect mitxpro_default | grep Gateway

3. Open the LMS container shell using `tutor dev exec -it lms bash` and install the required dependencies:

       pip install ol-social-auth
       pip install openedx-companion-auth

4. Create a `private.py` file at `edx-platform/lms/envs/{here}` and add the following configurations to allow additional OAuth providers

   ```
   from .production import AUTHENTICATION_BACKENDS, FEATURES, IDA_LOGOUT_URI_LIST, REGISTRATION_EXTRA_FIELDS

   FEATURES["ALLOW_PUBLIC_ACCOUNT_CREATION"] = True
   FEATURES["SKIP_EMAIL_VALIDATION"] = True

   REGISTRATION_EXTRA_FIELDS["country"] = "hidden"

   THIRD_PARTY_AUTH_BACKENDS = ["ol_social_auth.backends.OLOAuth2",]

   AUTHENTICATION_BACKENDS = list(THIRD_PARTY_AUTH_BACKENDS) + list(AUTHENTICATION_BACKENDS)

   IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://{Domain}:{PORT}/logout"])

   SOCIAL_AUTH_OAUTH_SECRETS = {
      "ol-oauth2": <mit_app_client_secret>  // you just copied from configure_instance command output
   }
   ```

5. Go to [`http://local.openedx.io:8000/admin/third_party_auth/oauth2providerconfig/add/`](http://local.openedx.io:8000/admin/third_party_auth/oauth2providerconfig/add/) and add a provider configuration:

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
    - `Client ID` and `Client Secret`: from record created by `configure_instance` when you set up MIT Application.
    - Other settings:

          {
             "AUTHORIZATION_URL": "http://{Domain}:{PORT}/oauth2/authorize/",
             "ACCESS_TOKEN_URL": "http://<MITApplication_GATEWAY_IP>:<PORT>/oauth2/token/",
             "API_ROOT": "http://<MITApplication_GATEWAY_IP>:<PORT>/"
          }

     where MITApplication_GATEWAY_IP is the IP from the `mitApplication_default` network from the first step. **Mac users**, use `host.docker.internal` for MITxApplication_GATEWAY_IP.

You should now be able to run some MIT Application management commands to ensure the service worker is set up properly:

   - `sync_courserun --all ALL` should sync the two test courses (if you made them).
      - `sync_courseruns --all ALL` in MITxPro
   - `repair_missing_courseware_records` should also work.

In the separate browser session, attempt to log in again. This time, you should be able to log in through MIT Application, and you should be able to get to the edX LMS dashboard. If not, then double-check your provider configuration settings and try again.

   - If you are still getting "Can't fetch settings" errors, **make sure** your Site is set properly.

**Optionally**, log into the LMS Django Admin and make your MIT Application superuser account a superuser there too.

## Other Notes

**Make sure your service worker account is active.** It's an easy checkbox to miss.

**Restarting** If you want to rebuild from scratch, make sure you `docker image prune`. It's also recommended to remove the Tutor project root folder - `tutor config printroot` will tell you where that is.
