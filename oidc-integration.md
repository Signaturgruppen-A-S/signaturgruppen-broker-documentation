---
title: OpenID Connect integration
layout: home
nav_order: 10
---

## OpenID Connect integration

### OpenID Connect Parameters

| Parameter | Description |
| --- | --- |
| client_id | Identifier of the client. |
| response_type | Authorization Code flow, Hybrid flow and Implicit flows are supported, as specified in [OIDC]. |
| redirect_uri | URI that the response will be sent to when authentication is finished. Must be from a list of preregistered URLs for this client. |
| scope | Space separated list of scopes that client is requesting authorization for. Note that openid must be included for all requests. |

Supported authorization request parameters are listed below. See [OIDC] for reference.

| Parameter | Description |
| --- | --- |
| nonce | String value used to associate a client session with an ID Token, and to mitigate replay attacks. Must be unique per request per client. |
| response_mode | Informs the Authorization Server of the mechanism to be used for returning Authorization Response parameters from the Authorization Endpoint. This use of this parameter is NOT RECOMMENDED with a value that specifies the same Response Mode as the default Response Mode for the Response Type used. Supported values query In this mode, Authorization Response parameters are encoded in the query string added to the redirect_uri when redirecting back to the Client. fragment In this mode, Authorization Response parameters are encoded in the fragment added to the redirect_uri when redirecting back to the Client. form_post In this mode, Authorization Response parameters are encoded as HTML form values that are auto-submitted in the User Agent, and thus are transmitted via the HTTP POST method to the Client, with the result parameters being encoded in the body using the application/x-www-form-urlencoded format. The action attribute of the form MUST be the Client's Redirection URI. The method of the form attribute MUST be POST. Because the Authorization Response is intended to be used only once, the Authorization Server MUST instruct the User Agent (and any intermediaries) not to store or reuse the content of the response. |
| state | ​Opaque value used to maintain state between the request and the callback. |
| request | This parameter enables OpenID Connect requests to be passed in a single, self-contained parameter and to be optionally signed and/or encrypted. It represents the request as a JWT whose claims are the request parameters. It is recommended for clients able to start flows with associated fees. This parameter can be made mandatory for a client in ADM-UI. See section 6 in [OIDC]. |
| request_uri | This parameter enables OpenID Connect requests to be passed by reference, rather than by value. The request_uri value is a URL using the https scheme referencing a resource containing a Request Object value, which is a JWT containing the request parameters. This parameter can be made mandatory for a client in ADM-UI. See section 6 in [OIDC]. |
| language | Sets the end-user language. Possible values are da (Danish) en (English) kl (Greenlandic) |
| max_age | Maximum Authentication Age. Specifies the allowable elapsed time in seconds since the last time the end-user was actively authenticated by NEB. If the elapsed time is greater than this value, NEB will attempt to actively re-authenticate the end-user. |
| prompt | Space delimited, case sensitive list of ASCII string values that specifies whether the Authorization Server prompts the end-user for reauthentication. Supported values are none (no ui for authentication) login (force authentication request) select_account (enable idp selection) If login is used, the end-user will be forced to complete the requested authentication flow. This can be used anytime it is required that the end-user completes the requested authentication flow, i.e. when requesting a signature from the end-user. If none is used, a request for automatic login based on the end-user session is requested. If the automatic login could not be honored, an error will be returned to the service. If select_account is used, the end-user will be allowed to select among the available identity providers for the given flow, even though the end-user has an active session with NEB. It does not trigger a forced reauthentication, so if the end-user selects the same identity provider for which he has an active session, he will be automatically reauthenticated. |

Supported identity provider parameters. See the “Signaturgruppen Broker Identity Providers” [NEB-IDP] for available options.

| Parameter | Description |
| --- | --- |
| idp_params | Identity provider parameters. Custom parameter supported by NEB to enable customization of the specific identity provider flows. See the “Signaturgruppen Broker Identity Providers” [NEB-IDP] for reference. The idp_params parameter value is represented in an OAuth 2.0 request as UTF-8 encoded JSON (which ends up being form-url-encoded when passed as an OAuth parameter). When used in a Request Object value, the JSON is used as the value of the idp_params member. |
| idp_values | Identity provider list. Space-separated string that specifies the idp values that the NEB is being requested to use for processing this authentication request, with the values appearing in order of preference. If not set, all possible identity providers configured for the client will be made available to the end-user. |

### Reauthentication / Step-up / Consent flows
As a general mechanism, NEB will always try to optimize the user experience and steps required from the end-user for an authentication flow. If a user has an active session with NEB and enters a new authentication flow, the existing session will under certain conditions be re-usable and only trigger additional actions needed from the end-user to fulfill the requested parameters.
As a general mechanism, NEB will look for any changes in requested scopes that may require additional input from the end-user, such as consents or other actions. This will be automatically handled by NEB.
Some identity providers support specific flow such as step-up under certain conditions. NEB will normally reuse an existing session and ignore any identity provider specific parameters in the authentication request if a session already exists. To force processing of the identity provider specific parameters, utilize the prompt=login parameter, which will trigger a re-authentication. This re-authentication might then support step-up if a previous session exists or any other identity provider specific handling, which will be described for each identity provider.

#### Requesting additional scopes
When requesting additional scopes for an existing user session only the required verifications and user-steps are invoked.
If the requesting client can request the additional scope, the existing session will be reused to issue a new session with the requested scopes. If the scope triggers end-user interaction, like a consent-flow, the user will be prompted for action, but the user will not have to reauthenticate unless required or requested.
Authentication Error Response
If the end-user denies the request or the end-user authentication fails, NEB informs the service provider (client) by using the error response parameters listed in this document or in the Identity Provider documentation [NEB-IDP].
Note, that the end-user will only be redirect back to the service provider (client) if the authorization request is valid. If the authorization request is invalid, the end-user will be presented a generic error response at NEB and will not be redirected back to the service provider.
The end-user is redirected (GET) or posted (POST) back to the redirect URL requested for the authentication request honoring the response_mode parameter.
NEB will utilize one of the error codes defined in the specification most appropriate to the error in question and will set the error_description to a NEB- or identity provider specific error code defined here or in the “Signaturgruppen Broker Identity Providers” [NEB-IDP],

##### Example response:

```
[client_redirect_uri]?error= access_denied&error_description=mitid_user_aborted&state=xyz
```

### Error codes
The error codes described here will be returned in the error_description parameter on error redirects or post backs to the redirect URI.
The complete list of error codes is available through the error codes API, found at the swagger endpoint. This endpoint is structured as a human readable JSON and can be consumed by humans in-browser or programmatically.
Errors relating to specific identity provider flows will be marked with the appropriate identity provider name.

### Scopes
Scopes are passed as a space separated list of string values in the scope authorization request parameter and determine the requested authorizations as well as the requested user claims.
A scope has the following functions (non-exhaustive):
Maps to a list of user claims for the ID token and UserInfo endpoint
Is mapped directly to the issued access token scope claim.
Some scopes trigger certain user flows or end-user actions such as required end-user consents.
Client must be allowed to use the scopes requested.
Examples are the identity provider specific scopes that maps to the list of identity provider specific user-claims issued in the flow, e.g. if the mitid scope is specified all non-empty mitid.* claims will be issued.
All requested scopes are mapped directly to the scope claim in the issued access- and service tokens, allowing scopes to be used in authorization contexts.
Some scopes trigger user-interaction or user-consents as they might map to claims which requires special actions or consents to be issued.
If a client is requesting a scope which is not allowed for this client, the entire flow will fail.

### Implicit flow and CORS
Signaturgruppen Broker supports the implicit flow (id_token) but does not support setting up any CORS origin rules for integrating clients and has no allowed CORS rules specified as default.
NEB does not encourage pure frontend-based integrations due to security concerns and has a very limited and restricted functionality available for frontend clients.
Frontend-based clients can request and complete the implicit flow but will have to pin the signing key certificate for verifying the ID token and will not be allowed accessing access tokens nor the Userinfo endpoint from the browser.
See the “Backend for Frontend Pattern” for reference for a recommended approach to securing your frontend and integration to NEB.

### Backend for Frontend Pattern
Backend for Frontend (BFF) is a pattern utilizing an API-based backend for your frontend which enables your frontend applications to be simplified, more secure and enables the full set of available features of your backend like back-channel logout, proper session handling, API proxying etc.
See Duendes blog about this pattern here:
For .Net core backends, see here for references: [Duende BFF framework](https://docs.duendesoftware.com/identityserver/v7/bff/)
