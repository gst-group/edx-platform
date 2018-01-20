OAuth Dispatch App (OAuth2 Provider Interface)
----------------------------------------------

The OAuth Dispatch app functions as a content-based router to the multiple
`OAuth2`_ provider implementations that exist within the platform. It was initially
created in order to transition the platform from using the (now unsupported and
deprecated) `Django OAuth Provider (DOP)`_ to the new Django-recommended
`Django OAuth Toolkit (DOT)`_ library. The app routes incoming OAuth2 REST
requests based on the value of the client_id field in the request. If the
client_id identifies an Application in DOT, then the request is routed to the
DOT library. Otherwise, the request is routed to the DOP library.

Once we fully execute the `transition plan from DOP to DOT`_, we will continue
to use and maintain this app as it also contains edx-specific customizations
that we have added over time. At that point, the app will no longer act as a
router but will retain its proxy functionality to DOT.

.. _OAuth2: https://tools.ietf.org/html/rfc6749
.. _Django OAuth Provider (DOP): https://github.com/caffeinehit/django-oauth2-provider
.. _Django OAuth Toolkit (DOT): https://github.com/evonove/django-oauth-toolkit
.. _transition plan from DOP to DOT: https://openedx.atlassian.net/wiki/spaces/OpenDev/pages/327778541/OAuth+2.0+Roadmap
