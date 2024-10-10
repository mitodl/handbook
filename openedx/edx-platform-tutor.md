---
parent: OpenedX
---
# Platform Development - Tutor

# Configure Open edX using Tutor

**NOTE:** {service} will be reffered to {Online|Pro} whichever service you are setting up

In order to create user accounts in Open edX and permit authentication from MITx {service} to Open edX, you need to configure MITx {service} as an OAuth2 provider for Open edX.

..

    These instructions should work for a Tutor Dev or Tutor Nightly deployment as well. Specify `--tutor-dev` instead of `--tutor` when running `configure_instance` so the URLs have a port on them.

## Preliminary Step

`pyenv` (and `pyenv-virtualenv`\ ) are highly recommended for managing local Python versions. `Use the instructions on their GitHub page to install if you haven't already installed it. <https://github.com/pyenv/pyenv>`_

You'll want to create at least a virtualenv for Tutor. As of this writing, Tutor uses Python 3.8.12 (in the LMS container at least); I have also successfully used 3.9.16. 3.11 has *not* worked for me.

Tutor Setup, Part One
---------------------

..

    Note that no hosts file changes are needed if you use the default `local.edly.io` domain - that's a real domain with a wildcard subdomain cname that points to 127.0.0.1.


To begin, you need to follow the `One-Click Installer <https://docs.tutor.overhang.io/quickstart.html>` instructions provided by Tutor. Do this with your Tutor virtualenv activated.

..

    Mac/Arm users should instead follow these instructions: `Running Tutor on ARM-based systems <https://docs.tutor.overhang.io/tutorials/arm64.html>` It's mostly the same steps that the quickstart does internally, with some changes to rebuild some of the images and flip some dependencies to use compatible images for Arm.


Once Tutor has bootstrapped itself and is available, create a superuser account:

.. code-block::

   tutor local do createuser --staff --superuser edx edx@example.org

Supply a password (the one used by devstack is `edx` so use that if you want to be consistent with it).

.. code-block::

   tutor local do createuser --staff mitx_{service}_serviceworker service@mitx.odl.local

Supply a password (this one doesn't matter for a local deployment, you won't ever actually use the account).

For best results, create two new courses within edX. The MITx {service} `configure_instance` command expects a couple of courses to exist in edX (because they come with the devstack package):

.. list-table::
   :header-rows: 1

   * - Course ID
     - Course Title
   * - course-v1:edX+DemoX+Demo_Course
     - Demonstration Course
   * - course-v1:edX+E2E-101+course
     - E2E Test Course


If you have a devstack instance handy, you can export these and import them into Tutor. Otherwise, just create them and make sure to set dates for the courses (they default to 2030 otherwise).

Finally, go here to create an access token for the service worker user: `<PROTOCOL>://<HOSTNAME>:<PORT>/admin/oauth2_provider/accesstoken/add/` The token can be anything, and the expiration date should just be today plus 10 years.

Tutor Setup, Part Two
---------------------

Note that some of these steps require editing the main configuration files for the production instance (which is also used for a local deployment). Most of the settings that need to be adjusted to get integration working are overridden by the default Tutor configuration, so you can't update them by setting `config.yml`. If you're using the development Tutor build, you'll likely need to edit `development.py` rather than `production.py` as necessary.

These steps will also disable the AuthN SSO MFE, so from here on you'll get normal edX authentication screens (if you're not being bounced to MITx {service}).


1. Get the gateway IP of the `mitxo{service}_default` Docker network:

        docker network inspect mitxo{service}_default | grep Gateway

2. Log into to edX using your superuser account, and make sure your session stays open. Sessions are pretty long-lived so this just means not closing the browser that you started the session in. (Part of this process will involve mostly breaking authentication so it's important that you are able to get into the admin.)
3. Stop Tutor: `tutor local stop`
4. Change into the configuration root for Tutor:

        cd "$(tutor config printroot)"

5. Create a `env/build/openedx/requirements/private.txt` with the required extensions:

        social-auth-mitxpro
        openedx-companion-auth

6. Edit the `env/apps/openedx/config/lms.env.yml` file and add:

        FEATURES:
            SKIP_EMAIL_VALIDATION: true

   The `FEATURES` block (should be at the top).
7. Edit the `env/apps/openedx/settings/lms/production.py` and/or `env/apps/openedx/settings/lms/development.py` settings file. (The former is used by a local instance, where the latter is used by both dev and nightly instances.)

   * Add to the end of the file:

      * `THIRD_PARTY_AUTH_BACKENDS = ['social_auth_mitxpro.backends.MITxProOAuth2']`
      * `AUTHENTICATION_BACKENDS.append('social_auth_mitxpro.backends.MITxProOAuth2')`
      * `IDA_LOGOUT_URI_LIST.append('http://mitx{service}.odl.local:<PORT>/logout/')` - there's an existing one of these around like 300 in production.py too.

    * Find and update:

      * `FEATURES['ENABLE_AUTHN_MICROFRONTEND'] = False` (defaults to True)
      * `REGISTRATION_EXTRA_FIELDS["terms_of_service"] = "hidden"` (defaults to required)

8. Build a new `openedx` image: `tutor images build openedx` (this will take a long time)
9. Run a Docker Compse rebuild: `tutor local dc build` (this should be pretty quick - it's likely not required, just doing it here for safety)
10. Restart Tutor: `tutor local start -d` (omit `-d` if you want to watch the logs)
11. Check your settings. There's a `print_setting` command that you can use to verify everything is set properly:

    * `tutor local run lms ./manage.py lms print_setting REGISTRATION_EXTRA_FIELDS`
    * `tutor local run lms ./manage.py lms print_setting AUTHENTICATION_BACKENDS`
    * `tutor local run lms ./manage.py lms print_setting FEATURES` - will print a lot of stuff
    * `tutor local run lms ./manage.py lms print_setting THIRD_PARTY_AUTH_BACKENDS`
    * If you do have weird errors or settings not showing properly, make sure you edited the right yaml files *and* that they're using the right whitespace (i.e. don't use tabs).

12. In a separate browser session of some kind (incognito/private browsing/other browser entirely), try to navigate to `http://local.edly.io:8000`. It should load but it should give you an error message. In the LMS logs, you should see an error message for "Can't fetch settings for disabled provider." This is proper operation - the OAuth2 settings aren't in place yet.
13. In the superuser session you have open, go to `http://local.edly.io:8000/admin`. This should work. If you've been logged out, you should still be able to get in. If you can't (for instance, if you're getting 500 errors), you will need to turn off `ENABLE_THIRD_PARTY_AUTH` in `FEATURES`, restart Tutor using `tutor local stop` and `start`, not using `reboot`, then try again.
14. Go to `http://local.edly.io:8000/admin/third_party_auth/oauth2providerconfig/add/` and add a provider configuration:

    * Enabled is checked.
    * Name: `mitx{service}`
    * Slug: `mitxpro-oauth2`
    * Site: `example.com`
    * Skip hinted login dialog is **checked**.
    * Skip registration form is **checked**.
    * Skip email verification is **checked**.
    * Sync learner profile data is **checked**.
    * Enable sso id verification is **checked**.
    * Backend name: `mitxpro-oauth2`
    * Other settings:

            {
                "AUTHORIZATION_URL": "\http://mitx{service}.odl.local:<PORT>/oauth2/authorize/",
                "ACCESS_TOKEN_URL": "\http://<MITX{service}_GATEWAY_IP>:<PORT>/oauth2/token/",
                "API_ROOT": "\http://<MITX{service}_GATEWAY_IP>:<PORT>/"
            }

     where MITX{service}_GATEWAY_IP is the IP from the `mitx{service}_default` network from the first step. **Mac users**, use `host.docker.internal` for MITX{service}_GATEWAY_IP.

    **NOTE:** Please note down the Client Id and Client Secret for MITx service integration

Other Notes
-----------

**Trying to set configuration settings via `tutor config` will undo the specialty configuration above.** If you need to make changes to the configuration, either manually edit the `env/apps/openedx/config/lms.env.yml` file or the `env/apps/openedx/settings/lms/production.py` file and restart your Tutor instance.

**Make sure your service worker account is active.** It's an easy checkbox to miss.

**Restarting** If you want to rebuild from scratch, make sure you `docker image prune`. It's also recommended to remove the Tutor project root folder - `tutor config printroot` will tell you where that is.

**Running Multiple Tutor Instances** If you want to run more than one Tutor instance, it's pretty important to specify the project root explicitly or you may end up with one instance trying to use config files from another and things getting confused from there. `See the Tutor documentation for this. <https://docs.tutor.overhang.io/local.html#tutor-root>`_ (A suggestion: configure aliases to the `tutor` command that run `tutor --root=<whatever>` so you don't have to rely on environment variables, especially if you keep multiple terminal sessions going.)
