---
title: OpenID Connect
layout: home
parent: Age Verification
nav_order: 1
---

# Age Verification using OpenID Connect (OIDC)

## Example OIDC request
The following example demonstrates a standard initialization of an AV flow using our OIDC interface.

```
https://pp.idbroker.eu/connect/authorize?client_id=[your client id]&redirect_uri=[your redirect uri]&response_type=code&scope=openid av:18&nonce=[your nonce]
```

## Example ID token (OIDC)
A successful flow will result in an ID token, which contains the relevant claims issued for the flow.

```
{
   "iss": "https://pp.idbroker.eu/op",
   "nbf": 1725009225,
   "iat": 1725009225,
   "exp": 1725009525,
   "aud": "[your client id]",
   "nonce": "[your-nonce]",
   "sub": "[random sub]",
   "auth_time": 1725009225,
   "idp": "eu_av",
   "transaction_id": "45..32",
   "idtoken_type": "av",
   "av:18": "true"
}
```
