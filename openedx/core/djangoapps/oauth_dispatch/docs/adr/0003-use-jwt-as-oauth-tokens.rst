3. Use JWT as OAuth2 Tokens
---------------------------

Status
------

Accepted

Context
-------

The edX system has external OAuth2 client applications, including edX Mobile apps
and external partner services. In addition, there are multiple edX microservices
that are OAuth2 Clients of the LMS. Some of these internal clients require `OpenID
Connect`_ features. Specifically, they make use of the `ID Token`_ extension to 
get user details from the LMS within the OAuth protocol. The ID Token allows OAuth2
clients to forward their tokens to other microservices without needing to reconnect
with the centralized LMS.

For example, the following request::

    curl -X POST -d "client_id=abc&client_secret=def&grant_type=client_credentials" https://courses.edx.org/oauth2/access_token/

returns::

    {
        "access_token": "xyz",
        "id_token": <BASE64-ENCODED-ID-TOKEN>,
        "expires_in": 31535999,
        "token_type": "Bearer",
        "scope": "profile openid email permissions"
    }

where the value of BASE64-ENCODED-ID-TOKEN decodes to::

    {
        "family_name": "User1",
        "administrator": false,
        "sub": "foo",
        "iss": "https://courses.edx.org/oauth2",
        "user_tracking_id": 1234,
        "preferred_username": "user1",
        "name": "User 1",
        "locale": "en",
        "given_name": "User 1",
        "exp": 1516757075,
        "iat": 1516753475,
        "email": "user1@edx.org",
        "aud": "bar"
    }

However, OpenID Connect is a large standard with many features and is not supported by
the DOT_ implementation.

.. _OpenID Connect: http://openid.net/connect/
.. _DOT: https://github.com/evonove/django-oauth-toolkit
.. _ID Token: http://openid.net/specs/openid-connect-core-1_0.html#IDToken

Decision
--------

Remove our dependency on OpenID Connect since we don't really need all its features
and it isn't supported by DOT. Instead, support `JSON Web Token (JWT)`_, which is a
simpler standard and integrates well with the OAuth2 protocol.

.. _JSON Web Token (JWT): https://jwt.io/

The simplest approach is to allow OAuth2 clients to request JWT tokens in place of
randomly generated Bearer tokens. JWT tokens contain user information, replacing
the need for OpenID's ID Tokens altogether.

So, an OAuth2 client requesting JWT tokens::

    curl -X POST -d "client_id=abc&client_secret=def&grant_type=client_credentials&token_type=jwt" https://courses.edx.org/oauth2/access_token/

would now receive::

    {
        "access_token": <BASE64-ENCODED-JWT>,
        "token_type": "JWT",
        "expires_in": 31535999,
        "scope": "read write profile email"
    }

where the value of BASE64-ENCODED-JWT decodes to what the BASE64-ENCODED-ID-TOKEN decodes to.

OAuth2 Clients that are not interested in receiving contents of the JWT could continue to
use the default Bearer token type::

    curl -X POST -d "client_id=abc&client_secret=def&grant_type=client_credentials" https://courses.edx.org/oauth2/access_token/

which returns::

    {
        "access_token": <RANDOM-UNIQUE-STRING>,
        "token_type": "Bearer",
        "expires_in": 36000,
        "scope": "read write profile email"
    }

Consequences
------------

* The long-term design of the system will be simpler by using simpler
  protocols and frameworks.

* During the transition period, there will be multiple implementations,
  which may result in confusion and a more complex system. The shorter
  we keep the transition period, the better.
