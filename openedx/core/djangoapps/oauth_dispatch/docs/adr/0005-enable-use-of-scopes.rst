OAuth2 Scopes
-------------

Application fields enforced by Authentication class:
scopes

Application fields enforced by Permission class:
user_org: User Organization
course_org: Course Organization



2. Security ramifications of rollout of OAuth Scopes:
When we enable OAuth Scopes and update some endpoints to enforce them, there exist other endpoints in the system that will be vulnerable to exploitation unless we have a rollout plan that prevents this security issue.

#3 - #5 below are ideas for how to address this.

3. Iteratively update other Microservices by introducing a new JWT signing key:
In order to not have a lock-step deployment across all of our IDAs and to protect against OAuth Applications misusing their unexpiring tokens for accessing unprotected endpoints on other IDAs, we can introduce a new JWT signing key to sign the new unexpiring tokens.  We would update other IDAs to honor the new JWT signing key only after they are updated to enforce OAuth Scopes.

oauth_dispatch's AccessTokenView.dispatch method is where we can decide which signing key to use, depending on the Application requesting the token.

The blue and purple services listed on this slide are the microservices we'll need to eventually update.  I have listed links to their repos in the comments section on that slide.

4. Only issue JWT tokens to Scope-requiring Applications
I didn't think about this on the call, however, a simpler (?) path forward that can alleviate the backward compatibility security implications is to issue only JWT tokens (not opaque OAuth2 Bearer tokens) to the new unrestricted Applications.  Advantages:
(1) We have fewer endpoints that support JwtAuthentication today so fewer exploitable OAuth2 endpoints.
(2) We have a common interface that all JWT supporting endpoints use so we can override any enforcements at a low-level (see #5).

5. Enforce Scopes at a low-level (e.g., within JwtAuthentication):
All of our JWT-supporting endpoints using the JwtAuthentication class.  We can override the implementation of authenticate_credentials to assure that all views will enforce scopes without needing to manually update each one.  This gives us high confidence that we won't miss any endpoints.

6. Use ClientCredentials grant for server-to-server bulk requests:
As mentioned, the OAuth2 Client Credentials grant type is the standard protocol for making server-to-server requests and simplifies your security design, eliminating the need for "Allowed Users" and such.  Open edX instances would create

(1) Authorization grant types for Applications that need to make calls on behalf of End Users,
(2) Client credential grant types for Applications that need to make bulk queries using a Service User.

A single Application can have multiple entries, one for each grant type, if needed.

7. Scaling bulk operations: Here's an example of where the filter is happening in the python code.  This may just be an oversight - but it's an example of the scaling types of feedback to expect in the code review process.