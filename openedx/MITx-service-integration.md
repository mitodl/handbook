---
parent: OpenedX
---

# Configure MITx online|pro as a OAuth provider for Open edX

There are 2 ways to setup edx-platform:
1. Tutor (Recomended) <https://github.com/mitodl/handbook/tree/master/openedx/edx-platform-tutor.md>
2. Devstack (Soon to be deprecated) <https://github.com/mitodl/handbook/tree/master/openedx/edx-platform-devstack.md>

**NOTE:** {service} will be reffered to {online|pro} whichever service you are setting up

## Run the configure_instance command

    docker-compose run --rm web ./manage.py configure_instance linux --gateway <ip> --tutor-dev --edx-oauth-client <client_id> --edx-oauth-secret <client_secret>

  Where `<ip>` is the IP from the first step and `<client_id>`, `<client_secret>` are the values from edx while setting up openedx setup you can also get them from `http://<EDX_HOSTNAME>:<PORT>/admin/third_party_auth/oauth2providerconfig/` and choose `mitx-oauth2`. (On macOS, specify macos instead of linux. You can also skip --gateway.) You will need to supply passwords for the MITx {service} superuser and test learner accounts. Make a note of the client ID and secret that it will print out at the end.

In MITx {service}:
* go to `/admin/oauth2_provider/application/` and create a new application with these settings selected:

  * `Redirect uris`: `http://<EDX_HOSTNAME>:<PORT>/auth/complete/mitxpro-oauth2/`

    * **[macOS users]** You will need redirect uris for both the local edX host alias and for `host.docker.internal`. This value should be:

      * `http://<EDX_HOSTNAME>:<PORT>/auth/complete/mitxpro-oauth2/`
      * `http://host.docker.internal:<PORT>/auth/complete/mitxpro-oauth2/`

    * **[Linux users]** You will need redirect uris for both the local edX host alias and for the gateway IP of the docker-compose networking setup for MITx {service} as found via `docker network inspect mitx-{service}_default`:
      * `http://<EDX_HOSTNAME>:<PORT>/auth/complete/mitxpro-oauth2/`
      * `http://<GATEWAY_IP>:<PORT>/auth/complete/mitxpro-oauth2/`

    * **[WSL 2 users]**: Use the URLs for macOS. You will also have to set `OPENEDX_IP` to `host-gateway` in your `.env` file to make this work. (Networking with WSL 2 works very differently, and the defaults won't work.)

    NOTE: `GATEWAY_IP` should be something like `172.19.0.1`.

  * `Client type`: "Confidential"
  * `Authorization grant type`: "Authorization code"
  * `Skip authorization`: checked
  * Other values are arbitrary but be sure to fill them all out. Save the client id and secret for later

## Configure Open edX to support OAuth2 authentication from MITx {service}

   * Go to `http:<EDX_HOSTNAME>:<PORT>///admin/oauth2_provider/application/` and either add or edit the `edx-oauth-app` entry.
   * Ensure these settings are set:

      * Name: `edx-oauth-app`
      * Redirect uris: `http://mitx{service}.odl.local:<port>/login/_private/complete`
      * Client type: `Confidential`
      * Authorization grant type: `Authorization code`
      * Skip authorization is checked.

   * Save `Client id` and `Client secret`.

Update your MITx {service} `.env` file. Set `OPENEDX_API_CLIENT_ID` and `OPENEDX_API_CLIENT_SECRET` to the values from the record you created or updated in the last step.
You should now be able to run some MITx {service} management commands to ensure the service worker is set up properly:

   * `sync_courserun --all ALL` should sync the two test courses (if you made them).
   * `repair_missing_courseware_records` should also work.

In the separate browser session, attempt to log in again. This time, you should be able to log in through MITx {service}, and you should be able to get to the edX LMS dashboard. If not, then double-check your provider configuration settings and try again.

   * If you are still getting "Can't fetch settings" errors, *make sure* your Site is set properly - there are three options by default and only one works. (This was typically the problem I had.)

Optionally, log into the LMS Django Admin and make your MITx {service} superuser account a superuser there too.
