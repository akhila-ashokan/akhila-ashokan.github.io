---
layout: post
title: Configuring  Keycloak Authentication on Airflow using Open ID
tags: [Airflow, Python]
color: brown
author: Akhila
excerpt_separator: <!--more-->
---

# Airflow comes with many authentication options. I thought I would document the steps I took to configure a custom provider, Keycloak, for Airflow authentication. 
This tutorial assumes you have Airflow configured on your system and know client credentials for Keycloak authentication. 

<!--more-->

* Set `rbac = True` in Airflow's config file (`airflow.cfg`)

This will be enable the Flask-Appbuilder UI (FAB) that Airflow uses for role-based access control (rbac) features. 

<!--more-->

* Restart the webserver through the CLI

`airflow webserver`

This should create the `webserver_config.py` file in the Airflow home directory.

<!--more-->

* Install [flask-oidc](https://flask-oidc.readthedocs.io/en/latest/) package in your virtual environment where Airflow is installed

`pip3 install flask-oidc`

<!--more-->

* Clone the [fab_oidc](https://github.com/ministryofjustice/fab-oidc) repo from Github in  the airflow home directory

<!--more-->

* Create `client-configuration.json` file with the following credential information and add to airflow home directory:

```
{
    "web": {
        "client_id": "WHATEVER_YOUR_CLIENT_ID_IS",
        "client_secret": "WHATEVER_YOUR_SECRET_IS",
        "auth_uri": "https://<domain>/auth/realms/myrealm/protocol/openid-connect/auth",
        "token_uri": "https://<domain>/auth/realms/myrealm/protocol/openid-connect/token",
        "userinfo_uri": "https://<domain>/auth/realms/myrealm/protocol/openid-connect/userinfo",
        "issuer": "https://<domain>/auth/myrealm/client_id",
        "redirect_uris": [
            "http://localhost:8080/oidc_callback"
        ]
    }
}
```

<!--more-->

* Modify `webserver_config.py` file with the following details

```
from flask_appbuilder.security.manager import AUTH_OID
import sys
sys.path.append('/absolute_path_to/airflow_home')
from fab_oidc.security import AirflowOIDCSecurityManager
AUTH_TYPE = AUTH_OID 
# Uncomment to setup Full admin role name 
# AUTH_ROLE_ADMIN = 'Admin'  
# Uncomment to setup Public role name, no authentication needed 
# AUTH_ROLE_PUBLIC = 'Public'  
# Will allow user self registration 
AUTH_USER_REGISTRATION = False  
# The default user self registration role 
AUTH_USER_REGISTRATION_ROLE = "Admin"  
OIDC_CLIENT_SECRETS = 'absolute_path_to/client-configuration.json' 
OIDC_VALID_ISSUERS = 'https://<domain>/auth/realms/myrealm' # can be found as iss in jwt access token
SECURITY_MANAGER_CLASS = AirflowOIDCSecurityManager 
OIDC_USER_INFO_ENABLED = True OIDC_SCOPES = ['openid', 'email', 'profile'] # verify with scope in jwt access token
OIDC_CLOCK_SKEW: 560 
OIDC_RESOURCE_CHECK_AUD: True 
OIDC_INTROSPECTION_AUTH_METHOD: 'client_secret_post' # verify with jwt access token 
OIDC_ID_TOKEN_COOKIE_SECURE = False # should be set to True in production environment
```

<!--more-->


* Set env variables in shell where program is running-change based on your specifications 

`export USERNAME_OIDC_FIELD='preferred_username`
<!--more-->
`export FIRST_NAME_OIDC_FIELD='given_name`
<!--more-->
`export LAST_NAME_OIDC_FIELD='family_name`

<!--more-->

* Restart airflow webserver

<!--more-->

* Through the Airflow CLI, create an admin user that has the same credentials (username, first name, last name, email) as your user on keycloak 

<!--more-->

* Visit the Airflow UI. You should now be redirected to the Keycloak login page. 


<!--more-->


## Helpful Resources 
<https://gist.github.com/thomasdarimont/145dc9aa857b831ff2eff221b79d179a>
<!--more-->
<https://stackoverflow.com/questions/29046866/basic-flask-openid-connect-example>
<!--more-->
<https://stackoverflow.com/questions/53477760/flask-oidc-with-keycloak-oidc-callback-default-callback-not-working>
<!--more-->
<https://stackoverflow.com/questions/40663585/flask-oidc-redirect-uri-value-being-overwritten-somewhere>
<!--more-->
<https://github.com/ministryofjustice/fab-oidc/issues/5>


