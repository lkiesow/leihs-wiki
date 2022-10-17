External Authentication Systems in _leihs_
===========================================

An external authentication system consist of an **HTTP service** and a
corresponding **configuration in _leihs_**. The role of the external service is to
assert that a user who desires to sign in to _leihs_ is who he or she claims to
be and forward this assertion to _leihs_. 

![Diagram External Authentication](./Authentication/external_authentication.svg)


## The Flow of Data during a Sign-in Process as an Example

The following steps are carried out during a successful sign in process when
using an _external authentication sytem_. 

1. A currently not signed in user provides her or his identity (either in the
   form of the `email`, `login`, or `org_id`) and requests to sign in via the
   _leihs_ UI.

2. Leihs evaluates this requests and offers available authentication methods for
   the particular user. Available authentication methods are:

    * Authentication systems specifically assigned to a user.
    * Authentication systems assigned to a group the user belogs to.
    * Authentication systems which `sign_up_email_match` field matches the user
      input of an unregistered user.

3. The user proceeds, for example, with an external authentication system.

4. _leihs_ prepares an JWT _authentication token_ and forwards the user to the
   external service via an HTTP redirect. The redirected URL will include a URL
   parameter `token`, holding the JWT data. For more details about the token,
   see the section [Authentication Token](#authentication-token) below.

5. The external authentication system will (in general first) _verify origin_
   and _validity_ of the token.

6. The external service performs authentication of he user.

7. The external service creates a further token which proves successful
   authentication and redirects the user back to _leihs_ using the path
   contained in the authentication token and providing the _success token_ via
   the URL parameter `token`.

8. _leihs_ will _verify origin_ and _validity_ of the token and creates a
   session for the user.

The procedure state above is the most general case. _leihs_ can shortcut steps
and follow simpler procedures in some cases. 


## Some Key Properties of External Authentication in _leihs_

* An external authentication system is bound to leihs with a public/private key
  pair (PKI) and via an URL. 

* For every external systems there is also a **public/private key pair** in _leihs_.  

* This allows the external authentication system to be controlled and run 
  independently of leihs. This setup (and the communication based purely 
  on redirects) **tolerates firewalls**. 

* The tokens send forth and back to leihs follow the [JWT](https://jwt.io/)
  standard.

* Leihs supports (currently exclusively) `ES256` keys, see also [External
  Authentication Keys](./external_authentication_keys).

* Users are **associated directly** or via **groups** to authentication systems in
  _leihs._ 


## Authentication Token

When using an external authentication system, _leihs_ will send data to the
external system by providing a [JWT token](https://jwt.io/) as URL parameter
`token`. For example, this would look like this:

    https://leihs-auth.example.com/?token=eyJhbGciOiJFUzI1NiJ9.eyJlbWF…

The payload of this _authentication token_ will look similar to this:

```json
{
  "email": "user@example.com",
  "login": null,
  "org_id": null,
  "exp": 1666039040,
  "iat": 1666038950,
  "server_base_url": "https://leihs.example.com",
  "return_to": "/",
  "path": "/sign-in/external-authentication/external-auth/sign-in"
}
```

Token fields:

* `email`, `login`, `org_id`: _leihs_ will try to find an existing user for the
  username or email typed in during log-in and will fill in these fields
  accordingly. If no user can be found, but `sign_up_email_match` matches, the
  matching value is provided as `email`. Empty fields are provided as `null`.
* `exp`: Timestamp specifying when this token will expire. The token provided by
  _leihs_ is valid for 90 seconds by default.
* `iat`: Timestamp specifying when this token was issued.
* `server_base_url`: The base URL of the server issuing this token. This should
  be the URL _leihs_ is deployed at.
* `path`: The path to which the user should be sent after successfully logging
  in and obtaining a _success token_

After verifying the token and authenticating the user, the external
authentication system will then generate a _success token_ and send it via
`token` URL parameter to _leihs_ at `server_base_url` + `path` like this:

    https://leihs.example.com/sign-in/external-authentication/external-auth/sign-in?token=eyJhbGciOiJFUzI1NiJ9.eyJlbWFpbCI6I…

The payload of this _success token_ should look similar to this:

```json
{
  "sign_in_request_token": "eyJhbGciOiJFUzI1NiJ9.eyJlbWFpbCI6Imxhcg…",
  "email": "user@example.com",
  "iat": 1666042616,
  "exp": 1666042736,
  "success": true
}
```

Token fields:

* `email`, `login`, `org_id`: Field used to identify the user.
* `exp`: Timestamp specifying when this token will expire.
* `iat`: Timestamp specifying when this token was issued.
* `success`: Must be `true` if the log-in was successful.
* `sign_in_request_token`: The _authentication token_ provided by _leihs_ .


## Configuration within an _leihs_ Instance

Authentication systems can be managed via the admin UI. Creating, updating and
deleting authentication systems requires the role of an _leihs system
administrator_.  Adding users and groups to an authentication system requires
the role of an _leihs adminstrator_.

See also [Roles](roles).


## Caveats

* The sign-in process via an external or internal authentication system in
  leihs **does not create or update properties of a user**. We highlight this
  because **_leihs_ prior to version 4 behaved differently**.

  The _leihs-admin_ service features an API to manage users and groups which
  should be used for this purpose.  It is possible (however not recommended) to
  combine these two functionalities in the same service.


## Implementations of Authentication-System Services

The following implementations can be used as examples or starting points for own implementations.

* [Azure Active Directory - leihs Authentication System](https://github.com/leihs/azure-active-directory-leihs-authentication-system)

  This authentication system uses the [OpenID Connect](https://de.wikipedia.org/wiki/OpenID_Connect) 
  feature of Azure Active Directory.

* [ZHdK Switch-AAI Shibboleth leihs Authentication-System](https://github.com/leihs/leihs-zhdk-switchaai-shibboleth-auth-system)

  [Switch-AAI](https://www.switch.ch/aai/) is a federated  "Authentication and Authorization Infrastructure" provided by Switch.
  Technology wise it is based on [**Shibboleth**](ci.zhdk.ch/cider-ci/commits/).

* [ZHdK AGW leihs Authentication-System](https://github.com/leihs/leihs-zhdk-agw-auth-system)

  AGW is the name of the internal authentication system at the ZHdK. The sign in process used by AGW
  is very similar to the one used by [**OAuth**](https://oauth.net/).

