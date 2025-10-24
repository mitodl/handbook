---
parent: OpenedX
---
# Integrating MIT xPro/MITx Online with Open edX using Tutor

| Application                  | Port | Domain       |
|------------------------------|------|---------------------|
| MIT xPro                     | 8053 | xpro.odl.local      |
| MITx Online                  | 8013 | mitxonline.odl.local |
| MITx Online(apisix/keycloak) | 9080 | mitxonline.odl.local |

## Prerequisite

To begin, you need to follow the [Installing Tutor for development](https://docs.tutor.edly.io/dev.html#open-edx-development) instructions provided by Tutor **for local development installations**. 

Once Tutor has bootstrapped itself and is available, create a superuser account:

    tutor dev do createuser --staff --superuser edx edx@example.org

Log in to your edX and MIT application as an admin and make sure the session remains active throughout the process. (Part of this process will involve mostly breaking authentication, so it's important that you are able to access the admin).

## Configure Open edX

   1. Go to [`http://local.openedx.io:8000/admin/oauth2_provider/application/`](http://local.openedx.io:8000/admin/oauth2_provider/application/).
   2. Add a new application with the following settings and save the `Client ID` and `Client Secret` later.:

| Application                   | Name                | Redirect URIs                                                                                                     | Client Type   | Authorization Grant Type | Skip Authorization |
|-------------------------------|---------------------|-------------------------------------------------------------------------------------------------------------------|---------------|--------------------------|--------------------|
| MIT xPro                      | xpro-oauth-app      | `http://xpro.odl.local:8053/login/_private/complete`<br>`http://xpro.odl.local:8053/_/auth/complete`              | Confidential  | Authorization code       | Checked            |
| MITx Online                   | mitxonline-oauth-app| `http://mitxonline.odl.local:8013/login/_private/complete`<br>`http://mitxonline.odl.local:8013/_/auth/complete`  | Confidential  | Authorization code       | Checked            |
| MITx Online (apisix/keycloak) | mitxonline-oauth-app| `http://mitxonline.odl.local:9080/login/_private/complete`<br>`http://mitxonline.odl.local:9080/_/auth/complete`  | Confidential  | Authorization code       | Checked            |

## Create a Service Worker and Access Token

1. Create a service worker.

       tutor dev do createuser --staff mitserviceworker mitserviceworker@mit.odl.local

2. Go to [`http://local.openedx.io:8000/admin/oauth2_provider/accesstoken/`](http://local.openedx.io:8000/admin/oauth2_provider/accesstoken/) and add a new token.
3. Set the following values:
   - User: `mitserviceworker` (or whatever you named the service worker)
   - Source refresh token: leave blank
   - Token: Add a random string (you will need this later)
   - Id token: leave blank
   - Application: leave blank
   - Expires: set to a future date
   - Scope: leave blank

## MIT xPro/MITx Online Setup

1. Get the gateway IP for the `EDX_APP`

       docker network inspect tutor_main_dev_default | grep Gateway

2. Set up your `.env` file. These settings need particular attention:

   - `OPENEDX_IP`: set to the gateway IP from the first step.
   - `EDX_IP`: set to the gateway IP from the first step.
   - `OPENEDX_API_BASE_URL`: `http://local.openedx.io:8000` (in case you are using tutor for local development)
   - `OPENEDX_SERVICE_WORKER_USERNAME`: `mitserviceworker` (or whatever you named the service worker in Open edX)
   - `OPENEDX_SERVICE_WORKER_API_TOKEN`: set to the access token you created for the service worker in Open edX
   - `OPENEDX_OAUTH_PROVIDER`: `ol-oauth2`
   - `OPENEDX_SOCIAL_LOGIN_PATH`: `/auth/login/ol-oauth2/?auth_entry=login`
   - `OPENEDX_API_CLIENT_ID`: set to the client id of the OAuth application you created in Open edX in the above steps
   - `OPENEDX_API_CLIENT_SECRET`: set to the client secret of the OAuth application you created in Open edX in the above steps
   - `OPENEDX_BASE_REDIRECT_URL`: `http://local.openedx.io:8000` (in case you are using tutor for local development)
   - `LOGOUT_REDIRECT_URL`: `http://local.openedx.io:8000/logout` (in case you are using tutor for local development)
   - For MITx Online:
     - `MITXONLINE_DOCKER_BASE_URL`:
       - set to `http://<GATEWAY_IP>:8013` (for MITxOnline)
       - set to `http://<GATEWAY_IP>:9080` (for MITxOnline with apisix/keycloak)
       - Note: MacOS users should use `host.docker.internal` instead of `<GATEWAY_IP>`.
     - `MITX_ONLINE_BASE_URL`:
       - set to `http://mitxonline.odl.local:8013` (for MITxOnline)
       - set to `http://mitxonline.odl.local:9080` (for MITxOnline with apisix/keycloak)

    Run the `docker-compose up -d` command after setting these values.

3. Create an OAuth2 application in the MIT xPro/MITx Online.
  - Option 1: Use the Django admin interface and add a new application at `/admin/oauth2_provider/application/` with the following settings:
    - Client id: save this for later
    - User: leave blank
    - Redirect URIs:
      - `http://local.openedx.io:8000/auth/complete/ol-oauth2/`
      - `http://<GATEWAY_IP>:8000/auth/complete/ol-oauth2/`
      - Note: Mac users should use `host.docker.internal` instead of `<GATEWAY_IP>`.
    - Client Type: `Confidential`
    - Authorization Grant Type: `Authorization code`
    - Client secret: save this for later
    - Name: `edx-oauth-app`
    - Skip Authorization: checked
    - Algorithm: No OIDC Support

  - Option 2: Use the `configure_instance` management command to set up the MIT Application and create an OAuth2 application in one step.

    - For `gateway_ip`, use the Gateway IP from the first step. (Specify `macos` or `linux` based on your OS. You can also skip --gateway.) The command will print `Client ID` and `Client Secret` that you will need further in the setup, so keep them safe. Alternatively, you can access the `Client ID` and `Client Secret` at `/admin/oauth2_provider/application/`
           
          docker-compose run --rm web ./manage.py configure_instance <linux or macos> --gateway <gateway_ip from above step> --tutor-dev

## Open edX Provider Configuration

1. Linux Users: Get the gateway IP of the Docker network.
   - For MIT xPro:

         docker network inspect xpro_default | grep Gateway
   - For MITx Online:

         docker network inspect mitxonline_default | grep Gateway

2. Install the required dependencies using one of the following:

   - **Option 1:** Open the LMS container shell using `tutor dev exec -it lms bash` and run:
     
         pip install ol-social-auth openedx-companion-auth

   - **Option 2:** Follow the [Tutor guide for installing extra requirements](https://docs.tutor.edly.io/configuration.html#installing-extra-xblocks-and-requirements).

3. Create a `private.py` file under `edx-platform/lms/envs/` and add the following configurations to allow additional OAuth providers

   ```
    ENABLE_MITXONLINE_OAUTH = False
    ENABLE_MITXPRO_OAUTH = False
    if ENABLE_MITXONLINE_OAUTH or ENABLE_MITXPRO_OAUTH:
        FEATURES["ALLOW_PUBLIC_ACCOUNT_CREATION"] = True
        FEATURES["ENABLE_COMBINED_LOGIN_REGISTRATION"] = True
        FEATURES["ENABLE_THIRD_PARTY_AUTH"] = True
        FEATURES["ENABLE_OAUTH2_PROVIDER"] = True
        FEATURES["SKIP_EMAIL_VALIDATION"] = True
        FEATURES['ENABLE_AUTHN_MICROFRONTEND'] = False
        FEATURES["ENABLE_UNICODE_USERNAME"] = True
    
        REGISTRATION_EXTRA_FIELDS["country"] = "hidden"
    
        THIRD_PARTY_AUTH_BACKENDS = [
            "ol_social_auth.backends.OLOAuth2",
        ]
    
        AUTHENTICATION_BACKENDS = list(THIRD_PARTY_AUTH_BACKENDS) + list(AUTHENTICATION_BACKENDS)
        if ENABLE_MITXONLINE_OAUTH:
            # Without APISIX
            # IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://mitxonline.odl.local:8013/logout"])
            # With APISIX
            IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://mitxonline.odl.local:9080/logout"])
    
        if ENABLE_MITXPRO_OAUTH:
            IDA_LOGOUT_URI_LIST = list(IDA_LOGOUT_URI_LIST) + list(["http://xpro.odl.local:8053/logout"])
        SOCIAL_AUTH_OAUTH_SECRETS = {
            "ol-oauth2": <mit_app_client_secret>  // client secret from the MIT xPro/MITx Online OAuth2 application
        }
   ```
4. Flip the appropriate flag to `True` based on the MIT application you are integrating with.
   - For MIT xPro:

         ENABLE_MITXPRO_OAUTH = True
         ENABLE_MITXONLINE_OAUTH = False

   - For MITx Online:

         ENABLE_MITXPRO_OAUTH = False
         ENABLE_MITXONLINE_OAUTH = True
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
    - `Client ID` and `Client Secret`: from the MITx Pro/MITx Online OAuth2 application you created earlier or copied from the output of the `configure_instance` command.
    - Other settings:
      - For MIT xPro:

            {
               "AUTHORIZATION_URL": "http://xpro.odl.local:8053/oauth2/authorize/",
               "ACCESS_TOKEN_URL": "http://<MITApplication_GATEWAY_IP>:8053/oauth2/token/",
               "API_ROOT": "http://<MITApplication_GATEWAY_IP>:8053/"
            }
      - For MITx Online (without apisix/keycloak):

            {
               "DISCOVERY_URL": "http://<MITApplication_GATEWAY_IP>:8013/.well-known/openid-configuration",
               "API_ROOT": "http://<MITApplication_GATEWAY_IP>:8013/"
            }
      - For MITx Online (with apisix/keycloak):

            {
               "DISCOVERY_URL": "http://<MITApplication_GATEWAY_IP>:9080/.well-known/openid-configuration",
               "API_ROOT": "http://<MITApplication_GATEWAY_IP>:9080/"
            }

     where `MITApplication_GATEWAY_IP` is the IP from the `mitApplication_default` network from the first step. **Mac users**, use `host.docker.internal` for `MITxApplication_GATEWAY_IP`.

6. Add another provider configuration with same settings as above but with the `Site` set to `example.com`
7. Confirm that the integration is working:
  - Create a new user in MIT xPro/MITx Online.
  - Confirm that:
    - For MIT xPro:
      - The user is created.
      - `courseware user` is created at `/admin/courseware/coursewareuser/`.
      - `open edx api auth` is created at `/admin/courseware/openedxapiauth/`.
    - For MITx Online:
      - The user is created.
      - `open edx user` is created at `/admin/openedx/openedxuser/`.
      - `open edx api auth` is created at `/admin/openedx/openedxapiauth/`.
  - The user is created in Open edX at `/admin/auth/user/`.
  - `user social auth` is created in Open edX at `/admin/social_django/usersocialauth/`.

In a separate browser session, attempt to log in again. This time, you should be able to log in through the MIT Application, and you should be able to get to the edX LMS dashboard. If not, then double-check your provider configuration settings and try again.

   - If you are still getting "Can't fetch settings" errors, **make sure** your Site is set properly.

**Optionally**, log into the LMS Django Admin and make your MIT Application superuser account a superuser there too.

## Troubleshooting

**Restarting** If you want to rebuild from scratch, make sure you `docker image prune`. It's also recommended to remove the Tutor project root folder - `tutor config printroot` will tell you where that is.
