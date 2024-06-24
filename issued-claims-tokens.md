---
title: Issued claims and tokens
layout: home
has_children: true
nav_order: 15
---

## User flow authentication result
A user flow results in one or more of the following tokens
ID token: Describing the authenticated end-user.
Access token: Providing access to configured endpoints on behalf of the end-user.
Refresh token (optional): Providing ability to maintain a long-lived session by re-acquiring access tokens using the refresh token.
Userinfo token (optional): Provides all user claims requested in a single JWT.
Transaction token (optional): Provides a self-contained sealed record of the transaction completed by the end-user.
Service token (Client Credentials Grant): Token issued directly to a service.
ID-, access-, service and transaction tokens comply with the [JWT] specification.
Access- and service tokens are not meaningful outside the audience of the token. Access tokens can be configured to include access and authorization for both internal and external REST APIs. Often the access token is used for accessing the UserInfo endpoint at NEB.
Refresh tokens are opaque and thus not meaningful outside the scope of NEB. See the OpenID Connect specification for reference on how to use the refresh tokens.
The userinfo token contains most of the ID token claims and all the claims issued by the Userinfo endpoint. This provides a signed format for all user-claims.
The transaction token will be available as a sealed (signed by a Nets eID Broker OCES3 organization certificate) record of the end-user completed transaction.
The transaction token response is a self-contained and cryptographically sealed record suitable for long-term storage and as a proof-of-transaction.
The transaction token includes the relevant authentication information, a unique transaction identifier as well as relevant transaction specific information like end-user approved text linked to the transaction.
The transaction token should only be requested if required for internal revision or similar requirements.

### ID token
Note, that the ID token does not include all issued user claims. The full list of user claims will be available from the UserInfo endpoint or in the Userinfo token.
ID tokens are issued from the Token endpoint or via the browser in Hybrid or Implicit flows.
ID tokens issued include the claims listed below with values as specified.

| Claim | Value |
| --- | --- |
| iss | Identifier for the issuer as an URL using https scheme. |
| neb_sid | Session ID. String identifier for a Session. This represents a session of a user agent or device for a logged-in end-user at a service provider. Different neb_sid values are used to identify distinct sessions at NEB. The neb_sid value need only be unique in the context of a particular issuer. |
| sub | NEB specific UUID representing the authenticated end-user. Primary end-user identifier. This identifier will be unique for each receiving organization for each associated identity. This means that all services under the same organization (defined in NEB admin-UI) will receive the same sub claim for the same identity (i.e. MitID identity). But services under different organizations will receive a different sub claim. More details are described in the Organization specific subject claims section. |
| aud | Audience(s) that this ID Token is intended for. This will be the ClientID of the receiving party. |
| exp | Expiration time on or after which the ID Token MUST NOT be accepted for processing. |
| iat | Time at which the JWT was issued. Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time. |
| auth_time | Time when the end-user authentication occurred. Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time. |
| nonce | String value used to associate a Client session with an ID Token, and to mitigate replay attacks. The value is passed through unmodified from the Authentication Request to the ID Token. |
| idp | Identity provider who issued the underlying identity. Refer to the specific identity providers for a list of possible idp values. |
| idp_environment | Optional available. See the different identity provider sections for specific details. If set, this specifies the environment for the identity provider specified in the idp claim. |
| identity_type | One of private professional test |
| transaction_id | Transaction ID. Unique identifier for the completed (end-user) transaction. Distinct identifiers are used to identify different transactions completed at NEB. |
| session_expiry | Absolute end-user session expiry represented as seconds elapsed since 00:00:00 UTC on 1 January 1970 (Unix timestamp) This value represents the time when the end-user session expires. Services may not use the issued authentication ticket from NEB to create new sessions after this time. |

Example of an ID token payload (decoded) from a MitID Identity Provider:

```
{
"sub"		: "bab646bb-8608-4ac7-ac42-cee4ad490600",
"mitid.uuid"		: "7027a386-aa7c-4dd6-93de-ebffd670f8b5",
"mitid.ial_identity_assurance_level"	: "MEDIUM",
"iss"		: "https://netsbroker.mitid.dk",
"jti"		: "5964f27b-7a7c-4f3d-99fe-aee934f03397",
"aud"		: "9ad129c2-0341-40e4-a184-b834272217dd",
"nonce"		: "3f0fc970-9727-4b3f-9f30-78793487ac7b",
"auth_time" 		: 1311261123,
"loa"       		: "https://data.gov.dk/concept/core/nsis/Low",
"amr"			: ["mitid.password"],
"identity_type"       		: "private",
"idp"       		: “mitid”,
"iat"       		: 1311290550,
"exp"      		: 1311291550,
"…."      		: ….,
}
```

Default expiry for ID tokens is 5 minutes.
The ID token is used when log-out is called for the end-user or when (re-)authenticating the end-user and forcing the same end-user to complete the transaction. The ID token is designed to include only the minimal required claim values to work as both browser-issued token in some scenarios, as well as logout token.

### Access token
The resulting access tokens authorizes the bearer on the behalf of the user. Unless otherwise configured for the client, the resulting access token provides authorization for the NEB UserInfo endpoint resulting in the full list of claims issued to the user for the authentication in question.
Depending on configuration, capabilities, roles, permissions and granted access for the client, the access token can authorize the client on behalf of the user to
Access specified APIs from NEB, like the Userinfo endpoint or the Privilege API.
Access internal APIs (i.e. internal to the Organization/Service in question)
Access external APIs
Expiry for access tokens is 3 hours.

### Service token (Client Credentials Grant)
If allowed, a service can retrieve a service token from the Token endpoint.
Services can get service tokens from the Token endpoint by authenticating directly using their client credentials and allows for issuing tokens for services directly.
Service tokens are used as bearer tokens exactly as access tokens. Service tokens are issued directly to a client (service) where an access token is issued to a client on behalf of an end-user.
Service tokens can have privileges and scopes just as access tokens, which defines the privileges and API’s that the service has been granted access to.
Service tokens are issued by using the Client Credentials Grant (section 4.4. in [OAuth]).
Default expiry for service tokens is 1 hour.

### Userinfo token
The userinfo token includes all issued user claims in a single signed JWT, including the full Userinfo endpoint response.
This allows the retrieval of all user claims directly from the Token endpoint in a signed format, signed with the same signing key as the ID- and access token.
The Userinfo token is requested by setting the scope value userinfo_token or can be set to always be returned for a specific client.

### Transaction token
The transaction token will contain additional claims based on the end-user identity provider and provided identity provider parameters.
It is not designed to replace the ID token and will most often contain sensitive information.
Transaction tokens are meant as a receipt for the completed end-user transaction. The token is signed by a special signing key and formatted to support long-term verification of the end-user transaction.
Many of the issued claims, are the same as found in the accompanied ID- or Userinfo token.
In this section all the claims that are always present in transaction tokens are specified. In addition, each identity provider will have its own set of claims that can be included depending on the context.
Transaction tokens will be set in the Token endpoint response, if configured for the client and if requested using the transaction_token scope.
An accompanying OCSP revocation check response for the signing certificate, will be set in the Token endpoint response, formatted as a UTF-8+Base64 encoded string (note that this response is currently under development).
Token endpoint response:

```
{
“id_token”: “AGGSSDDhY3Rpb…AFGGRh”,
…,
"transaction_token": “VHJhbnNhY3Rpb…BEYXRh”,
“transaction_token_ocsp_resp”: “VHJhbnNhY3Rpb2…UZXN0IERhdGE=”
}
```

Transaction tokens issued always include the following claims:

| Claim | Value |
| --- | --- |
| iss | Identifier for the issuer as an URL using https scheme. |
| sub | NEB specific UUID representing the authenticated end-user. Primary end-user identifier. |
| iat | Time at which the JWT was issued. Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time. |
| auth_time | Time when the end-user authentication occurred. Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time. |
| idp | Identity provider who issued the underlying identity. Refer to the specific identity providers for a list of possible idp values. |
| identity_type | One of private professional test |
| transaction_id | Transaction ID. Unique identifier for the completed (end-user) transaction. Distinct identifiers are used to identity different transactions completed at NEB. |
| recipient_info | Type: JSON. The top-level members of the recipient_info JSON object are: organization.number organization.name organization.country redirect_uri: The URL that the end-user is redirect to from NEB upon successful completing the transaction. The recipient_info parameter value is represented as UTF-8 encoded JSON. |
| transaction_actions | Type string or JSON list Contains the transaction actions completed by the end-user for the associated completed transaction (end-user flow). The possible values are described for each identity provider in [NEB-IDP]. Examples: { … "transaction_actions": "mitid.login" } { … "transaction_actions": [ "mitid.login", "mitid.cpr_match" ] } |
| spec_ver | 0.9 for this version. (Still under development) |

The receiving party should store both the transaction token and the OCSP response. Optionally, the signing certificate can be stored.

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

