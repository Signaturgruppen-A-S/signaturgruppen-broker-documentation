---
title: Technical reference
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

Supported identity provider parameters. See the “Nets eID Broker Identity Providers” [NEB-IDP] for available options.
| Parameter | Description |
| --- | --- |
| idp_params | Identity provider parameters. Custom parameter supported by NEB to enable customization of the specific identity provider flows. See the “Nets eID Broker Identity Providers” [NEB-IDP] for reference. The idp_params parameter value is represented in an OAuth 2.0 request as UTF-8 encoded JSON (which ends up being form-url-encoded when passed as an OAuth parameter). When used in a Request Object value, the JSON is used as the value of the idp_params member. |
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
NEB will utilize one of the error codes defined in the specification most appropriate to the error in question and will set the error_description to a NEB- or identity provider specific error code defined here or in the “Nets eID Broker Identity Providers” [NEB-IDP],

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
Nets eID Broker supports the implicit flow (id_token) but does not support setting up any CORS origin rules for integrating clients and has no allowed CORS rules specified as default.
NEB does not encourage pure frontend-based integrations due to security concerns and has a very limited and restricted functionality available for frontend clients.
Frontend-based clients can request and complete the implicit flow but will have to pin the signing key certificate for verifying the ID token and will not be allowed accessing access tokens nor the Userinfo endpoint from the browser.
See the “Backend for Frontend Pattern” for reference for a recommended approach to securing your frontend and integration to NEB.


### Backend for Frontend Pattern
Backend for Frontend (BFF) is a pattern utilizing an API-based backend for your frontend which enables your frontend applications to be simplified, more secure and enables the full set of available features of your backend like back-channel logout, proper session handling, API proxying etc.
See Duendes blog about this pattern here:
For .Net core backends, see here for references: https://docs.duendesoftware.com/identityserver/v5/bff/

## Organization specific subject claims
The subject claim, sub, issued in all ID tokens are the primary identity identifier and is always a unique organization specific GUID representing the identity for the receiving organization.
All services under the same organization, defined in the NEB administrative interface, will receive the same sub claim from the same identity.
This enables all service within an organization to map the same sub claim to the same identity.
If the same identity logs into a service under a different organization NEB will issue a different sub claim for this service/organization.
If a more global identifier is needed, the different identity providers will typically allow for identity provider specific identifiers to be requested such as the globally scoped MitID UUID – which will be available via the Userinfo endpoint.

### Choosing between ID token sub claim or identity provider specific identifiers
It is a common problem for an integrating service to decide how to map the incoming user from the incoming claims to a local user context.
Taking MitID as an example (replace with any specific identity provider), the choice is between registering a new user using the sub claim from the ID token or using the mitid.uuid claim, available from the Userinfo endpint.
Using the sub claim allows for minimizing the sensitivity of the received identifiers and information about the end-user, whereas the mitid.uuid claim is a global identifier providing a perhaps more useful identifier.
On a privacy note, it is always recommended to use the ID token sub, as this has the lowest impact on traceability of the end-user. On the other hand, mitid.uuid is perhaps required if a global identifier of the end-user is needed.
If only the ID token sub claim is needed, then a authentication flow does not even have to invoke the Userinfo endpoint after completion, thus minimizing both traffic and exchanged userinfo.
The recommendation is to consider whether the ID token sub can be used as the primary identifier for the end-users and only if this consideration determines that it is not sufficient, a move to the available global identifiers of the end-users should be considered.

## Security
This section will cover supported cryptographic algorithms, supported TLS versions, certificate pinning and other security related issues.
OpenID Connect provides a high level of security, but for some application require additional security hardening. This section will cover the available options.
All algorithms specified in this section is specified in [JWA].

### PKCE
Proof Key for Code Exchange by OAuth Public Clients (PKCE) is an extension to the Authorization Code flow to prevent certain attacks and to be able to securely perform the OAuth exchange from public clients.
This is prevalent and recommended when integrating to NEB from a mobile (Android and iOS) platform and allows the initiating app to be the only one who can retrieve the issued tokens, even though the client is a public client.
PKCE is fully supported. See [PKCE] for reference.

### Server certificate validation for TLS
When calling any of the NEB APIs or endpoints the integrity of the TLS certificate presented can be verified using the following checks
That the host name indicated in the certificate matches the host name of the APIs URL
That the presented certificate is issued by one of the publicly trusted CA’s listed in the documentation
That the certificate is within its validity period
That the signature in the certificate is valid
As an example, this can be used to verify the validity of the signing keys available from the Discovery endpoint.

### Nets eID Broker signing keys
Unless otherwise specified or configured all signed tokens issued by NEB will be signed by an HSM protected key at compliance level supporting the [FIPS 140-2] level 3 or equivalent.
All publicly available certificates are found through the OpenID Connect Discovery endpoint (TLS protected).
When signing tokens, NEB will use the algorithm ES256 (ECDSA using P-256 and SHA-256) or stronger.

#### Pinning signing certificates
The signing certificates used for all signed tokens will only be changed if required. This would include if the signing key is somehow compromised.
There will be support for getting upcoming signing certificate change notifications by email or by other mechanism via a bilateral agreement with NEB. It is expected, that signing certificates will be changed only if required due to security concerns.
Unless otherwise specified, the token signing certificates for will be self-signed with a very long time-to-live (10+ years).
The signing certificate for transaction tokens will be an OCES3 organization certificate, issued by the NemLog-In3 OCES3 CA. The DN of the transaction token signing certificate is available in the Environment section in this document, allowing pinning of the certificate DN.
The signing certificates will be made available through our documentation and through the agreed communication channels, allowing pinning of the specific certificates.

### Supported HTTPS/TLS versions
All endpoints will require TLS 1.2 or higher.
The general guidelines and requirements for MitID Brokers will be adhered to, as a minimum and updated continuously.

### JWT, JWS and JWE tokens
JWS (JSON Web Signature) and JWE (JSON Web Encryption) are the signed and encryption versions of JWT (JSON Web Token).
NEB always uses the JWS/JWE compact serialization format.
Note, that JWT tokens are always represented as either JWS or JWE.

### Supported signing and encryption algorithms for JWS and JWE tokens
If not otherwise specified, the following algorithms are supported for signing- and encryption operation of JWS and JWE tokens.
The listed options here, is the complete list of supported algorithms sending JWS or JWE tokens to NEB.
Note that the signing and encryption operations follow the standards for [JWS] and [JWE].
ECDSA signatures with ES256, ES384 and ES512.
RSASSA-PKCS1-V1_5 signatures with: RS256, RS384 and RS512.
RSASSA-PSS signatures (probabilistic signature scheme with appendix) with: PS256, PS384 and PS512.
HMAC signing algorithms: HS256, HS384, or HS512
RSASSA-PKCS1-V1_5 encryption with RSA1_5
RSAES OAEP encryption with RSA-OAEP
ECDH-ES encryption with ECDH-ES
Direct symmetric encryption with A128CBC, A256CBC, A128GCM, A256GCM

### Verification of tokens

#### ID token and Userinfo token
The basic checks are often implemented by the OIDC client library used for the integration.
To verify the authentication response the following steps from the OIDC specification are validated: .
The client MUST validate that the expected restrictions for amr, idp and identity_type are as expected.
The client MUST validate that the expected restrictions for identity provider specific claims are as expected. Example could be MitID claims: loa, ial and aal.

#### Access- and service token
Access- and service tokens are not verified by the client application, but by receiving services who use these tokens for authorization.
Access tokens issued by NEB conforms to the JWS specification and should be validated as an JWS Oauth2 Bearer token.
The service MUST perform standard JWS validation and the client MUST validate the expected values for the sub and iss claims.
The service MUST validate that the aud claim matches the required audience for the service.
The service MUST validate that the required scope claims are present in the token.
Alternatively, the service MAY call the NEB Privilege API (if the client is allowed) using the access token as authorization bearer token to receive the privileges (roles and permissions) for the end-user. In this scenario, the NEB Privilege API will perform all the required validation steps.

#### Transaction token
The expected Issuer Identifier MUST exactly match the value of the iss (issuer) claim.
The client MUST validate the signature of transaction tokens according to JWS [JWS] using the algorithm specified in the JWT alg Header Parameter. The client MUST use the keys provided by the Issuer (available via the Discovery endpoint).
The client MUST validate that the signing certificate is the certificate specified as the “Transaction token signing certificate” in the environments section in this document.
The iat Claim can be used to reject tokens that were issued too far away from the current time, limiting the amount of time that nonces need to be stored to prevent attacks. The acceptable range is client specific.
If the auth_time claim was requested, either through a specific request for this claim or by using the max_age parameter, the client SHOULD check the auth_time claim value and request re-authentication if it determines too much time has elapsed since the last end-user authentication.
The client MUST validate that the expected restrictions for acr, ial, amr, idp and identity_type are as expected.
Optionally, validate the OCSP response of the signing certificate, set as transaction_token_ocsp_resp in the token response (currently not implemented, will be released soon).
Optionally, validate the NONCE parameter by correlating the transaction_id found in the transaction token with the ID token, as the NONCE parameter is only set in the ID token.

### Verification of UserInfo endpoint response
Due to the possibility of token substitution attacks the UserInfo response is not guaranteed to be about the end-user identified by the sub (subject) element of the ID token. The sub claim in the UserInfo response MUST be verified to exactly match the sub claim in the ID token; if they do not match, the UserInfo response values MUST NOT be used.
The client MAY introduce additional security measures by pinning the TLS certificates or by requesting a signed response.

## Identity Providers

#### Multiple identity providers
It is possible to specify multiple identity providers for a single user flow through NEB.
A client can be configured for multiple identity providers as a default or specify more than one identity provider in the idp_values request parameter.
NEB will automatically let the user select the preferred identity provider for the current flow.
As an example, setting the idp_values parameter to “mitid nemid” enables the user to login with either MitID or NemID.

#### Identity Provider parameters
Specific settings supported by identity providers are set by including the idp_params request parameter in the authorization request.
The idp_params parameter value is represented in an OAuth 2.0 request as UTF-8 encoded JSON (which ends up being form-url-encoded when passed as an OAuth parameter). When used in a Request Object value, the JSON is used as the value of the idp_params member.
The top-level members of the idp_params request JSON object are:
[idp]: The idp in question (same value as the idp parameter).
The available options are found in the specific identity provider section in this document.
An example idp_params request is as follows:

```
{
"idp_params":
  {
  "mitid": {“reference_text”: “VHJhbnNmZXIgWCB0byBZ”},
  "nemid": {“remember_userid”: true, “code_app_trans_ctx”: “Transfer X to Y”}
  }
}
```

#### Resulting claims
The resulting authentication flow from any identity provider will result in at least an ID token and an access token. The basic user claims are always included in the ID token while the full list of user claims is available through the Userinfo endpoint.
See the ID token section in this document for details on the basic ID token claims.
Unless otherwise stated, user claims are available through the Userinfo endpoint. In each identity provider section, additional claims included in the tokens for these providers will be explicitly stated.

### Custom identity providers
It will be possible to setup integration to custom identity providers supporting OpenID Connect or SAML/WS-Federation based integrations.
This will allow NEB to handle authentications towards ex. Microsoft Azure, AD FS based setups, or any other internal or external identity provider based on either OpenID Connect or SAML.
Custom identity providers are handled like any other identity provider by NEB and allows integration with NEB SSO and other enterprise features.
