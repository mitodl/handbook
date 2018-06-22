# Touchstone integration via python-social-auth

### Pre-requisites
- Python packages
  - python-social-auth
  - social-auth-app-django
  - python3-saml
  
- OS requirements
  - libxmlsec1-dev
    - This should be added to `apt.txt` and `Aptfile`.
    - `https://github.com/heroku/heroku-buildpack-apt` should be 1st in the `buildpacks` section of `app.json`
    
- X509 certificate and private key
  ```shell
   openssl req -new -x509 -days 3650 -nodes -out saml.crt -keyout saml.key
   ```
   
- Touchstone identity provider (idp) metadata information
  - https://idp.mit.edu/idp/shibboleth
  
### Django metadata view and URL
- Create a public (no authentication) Django view to display SAML metadata in your app. For example:
  ```python
    from social_django.utils import load_strategy, load_backend
  
    def saml_metadata(request):
        """ Display SAML configuration metadata as XML """
        saml_backend = load_backend(
            load_strategy(request),
            "saml",
            redirect_uri=reverse('social:complete', args=("saml", )),
        )
        metadata, _ = saml_backend.generate_metadata_xml()
        return HttpResponse(content=metadata, content_type='text/xml')
  ```
- Create a url for the view
   ```python
    url(r'^saml/metadata/', saml_metadata, name='saml-metadata'),
   ```  

### Metadata configuration
- Using settings and environment variables, populate the required info for the SAML metadata:
```python
SOCIAL_AUTH_SAML_SP_ENTITY_ID = '&lt;Use the full metadata URL, ie https://myserver.edu/saml/metadata&gt>'
SOCIAL_AUTH_SAML_SP_PUBLIC_CERT = '&lt;x509 certificate with all line breaks removed&gt>'
SOCIAL_AUTH_SAML_SP_PRIVATE_KEY = '&lt;x509 key with all line breaks removed&gt>'

SOCIAL_AUTH_SAML_ORG_INFO = {
    "en-US": {
        "name": urlparse(SITE_BASE_URL).netloc,
        "displayname": "&lt;Enter any approproate value&gt;",
        "url": SITE_BASE_URL
    }
}
SOCIAL_AUTH_SAML_TECHNICAL_CONTACT = {
    "givenName": "&lt;Technical contact name&gt;",
    "emailAddress": "&lt;Technical contact email&gt;"
}
SOCIAL_AUTH_SAML_SUPPORT_CONTACT = SOCIAL_AUTH_SAML_TECHNICAL_CONTACT
SOCIAL_AUTH_SAML_ENABLED_IDPS = {
    "default": {
        "entity_id": "https://idp.mit.edu/shibboleth",
        "url": "https://idp.mit.edu/idp/profile/SAML2/Redirect/SSO",
        "attr_user_permanent_id": "urn:oid:1.3.6.1.4.1.5923.1.1.1.6", #EPPN
        "attr_username": "urn:oid:2.16.840.1.113730.3.1.241", #NAME
        "attr_email": "urn:oid:0.9.2342.19200300.100.1.3", #EMAIL
        "x509cert": "&lt;ds:X509Certificate value without linebreaks from Touchstone idp metadata&gt;",
    }
}

SOCIAL_AUTH_SAML_SECURITY_CONFIG = {
    "wantAssertionsEncrypted": True, # Mandatory for Touchstone
    "requestedAuthnContext": False  # If false, enables web certificate option in Touchstone
}

```


### Social auth pipeline: add a profile for the user

- Add a step to the social-auth pipeline for creating a user profile, for example:

   ```python
   # settings.py
   SOCIAL_AUTH_PIPELINE = (
    .....
    # require a profile if they're not set via SAML
    'authentication.pipeline.user.require_profile_update_user_via_saml',
    .....
    )
    
    # authentication/pipeline/user.py
    @partial
    def require_profile_update_user_via_saml(
            strategy, backend, user=None, is_new=False, *args, **kwargs
    ):  # pylint: disable=unused-argument    
        if backend.name != SAMLAuth.name or not is_new:
            return {}
    
        profile, _ = Profile.objects.update_or_create(
            user=user,
            defaults={
                'name': user.get_full_name()
            },
        )
    
        return {
            'user': user,
            'profile': profile,
        }
   ```

### Register your app with Touchstone

Contact touchstone-support@mit.edu and request to register your app.  

Include the x509 certificate string along with the following:
```
Web server host name: URL of your SAML metadata
Contact email address: odl-discussions-support@mit.edu
Organization Name: MIT Office of Digital Learning
Organization URL: https://openlearning.mit.edu  
```
