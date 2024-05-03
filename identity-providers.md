---
title: Identity providers
layout: home
nav_order: 25
---

## Identity Providers

### Multiple identity providers
It is possible to specify multiple identity providers for a single user flow through NEB.
A client can be configured for multiple identity providers as a default or specify more than one identity provider in the idp_values request parameter.
NEB will automatically let the user select the preferred identity provider for the current flow.
As an example, setting the idp_values parameter to “mitid nemid” enables the user to login with either MitID or NemID.

### Identity Provider parameters
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
  "mitid": {“reference_text”: “VHJhbnNmZXIgWCB0byBZ”}
  }
}
```

### Resulting claims
The resulting authentication flow from any identity provider will result in at least an ID token and an access token. The basic user claims are always included in the ID token while the full list of user claims is available through the Userinfo endpoint.
See the ID token section in this document for details on the basic ID token claims.
Unless otherwise stated, user claims are available through the Userinfo endpoint. In each identity provider section, additional claims included in the tokens for these providers will be explicitly stated.
