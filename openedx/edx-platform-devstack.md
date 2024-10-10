---
parent: OpenedX
---
# Platform Development - Devstack

# Configure Open edX using Devstack

**NOTE:** {service} will be reffered to {Online|Pro} whichever service you are setting up

In order to create user accounts in Open edX and permit authentication from MITx {service} to Open edX, you need to configure MITx {service} as an OAuth2 provider for Open edX.

## Setup Open edX Devstack

Following steps are inspired by `edx-devstack <https://github.com/edx/devstack>`.

## Add `/etc/hosts` alias for Open edX

If one doesn't already exist, add an alias to `/etc/hosts` for Open edX. We have standardized this alias
to `edx.odl.local`. Your `/etc/hosts` entry should look like this::

`127.0.0.1  edx.odl.local`

## Clone edx/devstack

.. code-block:: shell

    git clone https://github.com/edx/devstack
    cd devstack
    make requirements
    make dev.clone

## Pull latest images and run provision

.. code-block:: shell

    make pull
    make dev.provision

## Start your servers

.. code-block:: shell

    make dev.up

## Stop your servers

.. code-block:: shell

    make stop

## Setup social auth

## Install `social-auth-mitxpro` in LMS

### There are two options for this:

#### Install via pip

.. code-block:: shell

    pip install social-auth-mitxpro

#### Install from local Build

- Checkout the `social-auth-mitxpro <https://github.com/mitodl/social-auth-mitxpro>` project and build the package per the project instructions
- Copy the `social-auth-mitxpro-$VERSION.tar.gz` file into devstack's `edx-platform` directory
- In devstack, run `make lms-shell` and within that shell `pip install social-auth-mitxpro-$VERSION.tar.gz`

  - To update to a new development version without having to actually bump the package version, simply `pip uninstall social-auth-mitxpro`, then install again

## Install `openedx-companion-auth` in LMS

### There are two options for this:

#### Install via pip

.. code-block:: shell

    pip install openedx-companion-auth

#### Install from local Build

- Checkout the `openedx-companion-auth <https://github.com/mitodl/open-edx-plugins/tree/main/src/openedx_companion_auth>`\_ project and build the package per the project instructions
- Copy the `openedx-companion-auth-$VERSION.tar.gz` file from the `dist` folder into devstack's `edx-platform` directory
- In devstack, run `make lms-shell` and within that shell `pip install openedx-companion-auth-$VERSION.tar.gz`

  - To update to a new development version without having to actually bump the package version, simply `pip uninstall -y openedx-companion-auth`, then install again

In Open edX (derived from instructions `here <https://edx.readthedocs.io/projects/edx-installing-configuring-and-running/en/latest/configuration/tpa/tpa_integrate_open/tpa_oauth.html#additional-oauth2-providers-advanced>`):

* `make lms-shell` into the LMS container and ensure the following settings are set in `/edx/etc/lms.yml` if you are using Juniper or a more recent Open edX release, otherwise they should be in `/edx/app/edxapp/cms.env.json`:

    .. code-block:: yaml

        FEATURES:
            ALLOW_PUBLIC_ACCOUNT_CREATION: true
            ENABLE_COMBINED_LOGIN_REGISTRATION: true
            ENABLE_THIRD_PARTY_AUTH: true
            ENABLE_OAUTH2_PROVIDER: true
            SKIP_EMAIL_VALIDATION: true
            REGISTRATION_EXTRA_FIELDS:
            country: hidden
        THIRD_PARTY_AUTH_BACKENDS:
        - social_auth_mitxpro.backends.MITxProOAuth2

* ``make lms-restart`` to pick up the configuration changes
* Login to django-admin (default username and password can be found `here <https://github.com/openedx/devstack#usernames-and-passwords>`_), go to ``http://<EDX_HOSTNAME>:18000/admin/third_party_auth/oauth2providerconfig/``, and create a new config:

  * Select the default example site
  * The slug field **MUST** match the the backend's name, which for us is `mitxpro-oauth2`
  * Check the following checkboxes:

    * Enabled
    * Skip hinted login dialog
    * Skip registration form
    * Sync learner profile data
    * Enable SSO id verification
  * Set Backend name to: ``mitxpro-oauth2``

  * In "Other settings", put:

    .. code-block:: json

        {
            "AUTHORIZATION_URL": "http://<LOCAL_MITX_{service}_ALIAS>:<PORT>/oauth2/authorize/",
            "ACCESS_TOKEN_URL": "http://<EXTERNAL_MITX_{service}_HOST>:8013/oauth2/token/",
            "API_ROOT": "http://<EXTERNAL_MITX_{service}_HOST>:8013/"
        }

  * ``LOCAL_MITX_{service}_ALIAS`` should be your ``/etc/hosts`` alias for the mitx{service} app
  * ``EXTERNAL_MITX_{service}_HOST`` will depend on your OS, but it needs to be resolvable within the edx container

    * Linux users: The gateway IP of the docker-compose networking setup for mitx{service} as found via ``docker network inspect mitx-{service}_default``
    * OSX users: Use ``host.docker.internal``

  * Save the configuration.

    **NOTE:** Please note down the Client Id and Client Secret for MITx service integration
