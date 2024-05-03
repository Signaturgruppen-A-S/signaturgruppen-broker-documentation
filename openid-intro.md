---
title: OpenID Connect Intro
layout: home
nav_order: 4
---

# OpenID Connect Introduction

## Changelog

### 03-05-2024
* Created this online version.
* Renamed to Signaturgruppen Broker

## Introduction
This document aims at introducing OpenID Connect with respect to the integration to Signaturgruppen Broker. Simple guides and the most basic functionality will be described in this document.

Refer to the “Nets eID Broker Technical Reference” for a more in-depth technical documentation.

The intended audiences are IT developers and IT architects.

General information, online demonstration, documentation (including newest version of this document) and example code is found at .

## What is OpenID Connect?

OpenID Connect (OIDC) is an identity protocol that extends OAuth 2.0 to enable identity verification. It supports web, mobile, and single sign-on (SSO) applications.

### Key Features:
- Uses OAuth 2.0 for authorization and authentication.
- Provides standardized identity tokens in JWT format.
- Focuses on authentication rather than authorization.

### OIDC Tokens

1. **ID Token:** Contains information about the end-user authentication in JWT format.
2. **Access Token:** Grants access to user resources based on the requested scopes. Used to query the Userinfo endpoint for more user details.

**Note:** Tokens are signed by the OIDC provider.

### OIDC Flows

Select an OpenID Connect flow based on your application's requirements:

1. **Implicit Flow:** Ideal for SPA clients, tokens are directly returned to the relying party.
2. **Authorization Code Flow:** The recommended flow for most applications, exchanging an authorization code for tokens via the token endpoint.
3. **Hybrid Flow:** Combines implicit and authorization code flows. Returns the ID token directly but exchanges the authorization code for the access token.

### OIDC Flows

The choice of OpenID Connect flow depends on the type of application and its security requirements. There are three common flows.

Implicit Flow: In this flow, commonly used by SPA’s, tokens are returned directly to the RP in a redirect URI. Note that NEB only allows for ID tokens in this scenario. This scenario enables frontend-only applications to authenticate the end-user and avoid any (CORS) API call to the OIDC provider.

Authorization Code Flow: The recommended OIDC flow for most applications. The client application receives an authorization code which then is exchanged to the ID- and access tokens (and optionally additional tokens) via the Token endpoint server to server using a confidential client (entails a client secret).

Hybrid Flow: Combining Implicit and Authorization Code flows, here, the ID Token is returned directly to the RP, but the access token is not. Instead, an authorization code is returned that is exchanged for an access token. As an example, the OWIN framework in .Net requires the receival of the ID token, and thus does not support the Authorization Code Flow.

## Basic OIDC Flow

To get started:

1. Open this URL in a browser:

```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=0a775a87-878c-4b83-abe3-ee29c720c3e7&redirect_uri=https://openidconnect.net/callback&response_type=code&scope=openid mitid
```

2. Log in using a MitID test user. Instructions for creating a test user are provided on the login screen.

3. Follow the provided steps to exchange the authorization code for tokens and verify their contents.

4. Retrieve the access token from the response and manually query the Userinfo endpoint using Postman or another API client. Refer to the readme in the .NET Core demo for endpoint details.

## Signing Authentication Requests

### Signing Request Parameters

OpenID Connect's Request Object can encapsulate most parameters in a signed (and optionally encrypted) JWT. This approach is optional but recommended for transactional services like MitID.

NEB supports both Request Object by value and by reference (using `request_uri`). Both `client_id` and `redirect_uri` must be included as query parameters when using signed requests.

**Note:** Signing requires a configured client secret.

See [https://docs.duendesoftware.com/identityserver/v7/tokens/jar/](Duende Identity Server JAR spec)

### Example: Request Object by Reference

Example of a signed authentication request:

```
client_id: bcc7595b-2ed0-445d-a507-42ca21382877

request: eyJhbGciOiJIUzI1NiIsImtpZCI6bnVsbCwidHlwIjoiSldUIn0.eyJjbGllbnRfaWQiOiJiY2M3NTk1Yi0yZWQwLTQ0NWQtYTUwNy00MmNhMjEzODI4NzciLCJyZWRpcmVjdF91cmkiOiJodHRwczovL2xvY2FsaG9zdDo1MDE1L3NpZ25pbi1vaWRjIiwicmVzcG9uc2VfdHlwZSI6ImNvZGUiLCJjb2RlX2NoYWxsZW5nZSI6ImlaWFppX3RMbWhrc2FSZVdsR0JqbERSU1FuLXdrWUVsdXVsak0zbDZRSkUiLCJjb2RlX2NoYWxsZW5nZV9tZXRob2QiOiJTMjU2IiwicmVzcG9uc2VfbW9kZSI6ImZvcm1fcG9zdCIsIm5vbmNlIjoiNjM3NTcwMjAwNDg3MjI4MTg2LlpqUmpOV1V4TWpjdE1tWXdZeTAwTjJZeExXRXlZelF0T0RBek5qY3hZbVV4T0dFek16UTJOak13Wm1F...
```

### NEB Allows Any Redirect URI for Signed Requests

When using signed requests, clients can specify any redirect URI, allowing dynamic protocol usage and avoiding the whitelist approach.

## Signing Token Endpoint Requests with Private Key JWTs

The OIDC specification recommends client authentication via asymmetric keys. Instead of transmitting the shared secret over the network, the client generates a JWT signed with its private key.

### Example

A private key JWT is prepared and signed with the following format

```
{
  "jti": "ec454782-93c1-4160-8e46-502532df52f2",
  "sub": "bcc7595b-2ed0-445d-a507-42ca21382877",
  "iat": 1619296461,
  "nbf": 1619296461,
  "exp": 1619296581,
  "iss": "bcc7595b-2ed0-445d-a507-42ca21382877",
  "aud": ""
}
```

And heres a curl example of the post request:

```
curl --location --request POST '' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=authorization_code' \
--data-urlencode 'client_id=asd...ca21382877' \
--data-urlencode 'redirect_uri=<redirectUri>' \
--data-urlencode 'code=35CA4BC2A0F5DB9A....1BEF00150EEA44' \
--data-urlencode 'client_assertion=...XahMP78S7DdgwZEvVa4vA3usNsJg' \
--data-urlencode 'client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer'
```

More details for .Net can be found here: https://docs.duendesoftware.com/identityserver/v5/tokens/authentication/jwt/.

## Encrypting Requests

Using the signed request object parameter allows for encryption using a JWE. Follow these steps:

1. Retrieve the request encryption certificate from the JWKS endpoint.
2. Cache the certificate unless updated due to security concerns.
3. Create a JWE from the signed request object JWS.

Supported algorithms can be found in JWE implementations.

### Example of encrypted request

Example of encrypted request jwt (request parameter):

Header of JWE:

```
{

"alg": "http://www.w3.org/2001/04/xmlenc#rsa-oaep",
"enc": "A128CBC-HS256",
"kid": "9660F7AFFF397D67B77C92ED96F22A5A151F14C6",
"typ": "JWT"

}
```

JWE (encrypted request object):

```
eyJhbGciOiJodHRwOi8vd3d3LnczLm9yZy8yMDAxLzA0L3htbGVuYyNyc2Etb2FlcCIsImVuYyI6IkExMjhDQkMtSFMyNTYiLCJraWQiOiI5NjYwRjdBRkZGMzk3RDY3Qjc3QzkyRUQ5NkYyMkE1QTE1MUYxNEM2IiwidHlwIjoiSldUIn0.jiSxRegKLPFGeWoBkgBxqlJRSoI051p0QeS5pYFhEIoP1He_TtCYDkhB86ojMNp786ygYuhdGr3IdaJVoXmezA-cGynk555h9LLeLRot7QPxJAht_vwkY8rGDvi51rfg1hGAQEJBonbN0MhA7wpRvZej4pIX10QSeoLfi0oA3yIn-g0siRkVWKIOxJVQHF_-Cazvq-mgBDjSnge1j5Ad6hbFPjmdZ_EzZ-nCjiv1TvUCmEATTclhBpu3kkGuUuz6BGojGlCdEqOBM7SEpvfa-fBqGqAaPPt2M7qjb4wquL4ijzhbFLlAwnMSfByQSqVZeSb69ykYFu5fP3It_aXyNg._GEz-eukkuTNgJ77I3SWeA.JQ20OQDkKbDtw7g79Xlnh_CaT-QeYWJybIvztB-rWmwFZpitahPsXEAcl_vqkfEwILkgsuhCTSk8YOD2uGutDwLHeK7_u6Uh45hcvRmDAFyXp5fPvx5YZCmJRjfvWhIOTy6PxtXgnvBrRvfPypCoajeH2rX3946DXX8PeBXxk1gBEtj2Wlw-7dqXZsR8SAXzYglePu6P--QsbkTQGqzvjCwT_FaTcwbGLIB4f13DtodyBMOnTFwvfFkqyNCn_Aynw0Bzr992VyYMh9p_ADIbLdL1jTnNuVz7CQpmz14uHOkR--BogcIbe9o0bl2YYDZ-eRf9NED5Q3WoEvMswid2RZvjIWMgyA1HHTyz1TS1x5gzIO6RxrUVPjpfJg3eAT_efw2iNtU7PQiINoB1ohIgdrsKN4RbgsmnsIo2hj8AH1Gg0VkjEr79_oZJTOjCKW_Va4fTDyBNAABzP94dBeX8wBqrTRGwZff1nUhDH8PR3cxJ7VfrPo6XlNZWdRj9B4xJ1nHei51uJZPOFVnvw2uqjLDMhNnzuUT0TI6Ft-sJPDOe6z9FT79MqbWhew6ayeykZhE_Eg-MPv4NPq_B8rlT5C6oPJGQd4w14V0zKdJ2_GI7a0eCdLjLNufijCfcWNaMqUQKP1NCDOEDme2S8yOnRei9sIyd-UbNRtSjVgcsbJZD27HJhLy2BTeNN35_UftK-CpkwX_b3MzvQDtSZcFvssfY5YQRJ2wPjrrMkKcav2ywFpENB1e8tP9Vsw6nfeyzRWuTmixJMSODVWWDS6tyIe2w1rU3AeKmj7VUglG1WzjZDlGKosQLsW2gsWx6Jttq2Fgzw_wCBRWNpBJZ7Evoi560EbLADTnEDmsJpcJXKQNmKbfB4kb8BLRRq4Z-TNRgWHkDx4t9HWmRYCjXWPs4SFkXqAIgO5uNBU-0EzaWu0BgfvlZinM5h0wYUeQS_ADmCugbmPCNHjSf-RTIYocfQOS8ARtzjFPedVNLSDM0oiIvY5jP3E9mYelGo2C8JUrL-ZISLg7HssUKhrr2euE-2E_dY3M22DsjD0Z33ROiCf4jJxoS6sKFxJXV5BfSYcK5ZorBC65_Uszcx4DJepSp0BGPOjriizcYj3aC0z1lXB9EiS_feyMzUfnhK9euq8z2yNpbS3XCJmwIkvlbOxX38bHqAKLdDwgNoh_y2IJs6uGTMPKxxHqIaNT8FaekRYlQYO3C08eQQWBBbnmVdrLSrMF2Th07KUlss07sSy9EgBiZaEun_v_qSkoxcM9Edqnw4GnCM0ztR1sbmhqjJF68nwqcv3-bR2CuLIOzVOFBtiY.B9MljtvdtMz5E2RMBzjiCg
```

header of decrypted JWS:

```
{
"alg": "HS256",
"kid": null,
"typ": "JWT"
}
```

Decrypted JWS request object:

```
eyJhbGciOiJIUzI1NiIsImtpZCI6bnVsbCwidHlwIjoiSldUIn0.eyJjbGllbnRfaWQiOiI4NzlkZGQ2Mi0zMWM2LTQxNWEtYWYzMi1jZGVjNDc5MTdkZGQiLCJyZWRpcmVjdF91cmkiOiJodHRwczovL2xvY2FsaG9zdDo1MDE1L3NpZ25pbi1vaWRjIiwicmVzcG9uc2VfdHlwZSI6ImNvZGUiLCJjb2RlX2NoYWxsZW5nZSI6Ind5enF3ZjBxUFEycmxlNVNGMzVLYldJWDVvWmwzQmtiRGxDUy1PZ2ZMWVEiLCJjb2RlX2NoYWxsZW5nZV9tZXRob2QiOiJTMjU2IiwicmVzcG9uc2VfbW9kZSI6ImZvcm1fcG9zdCIsIm5vbmNlIjoiNjM3ODAwODE4NTI5MjY4NTA4Lk4yVmpZMlUzTkRBdE5USTVNQzAwTlRRMkxUbGpaREV0TVRkbU56WXpOVFUyT1RaalpqWXlNMkUwWVRrdE1USXhPQzAwWVRGaExXRXhNRGd0TVRZMk5tTXhZbU15Wm1abCIsInNjb3BlIjoib3BlbmlkIiwiaWRwX3ZhbHVlcyI6IiIsImlkcF9wYXJhbXMiOiJ7XCJtaXRpZFwiOntcImVuYWJsZV9hcHBfc3dpdGNoXCI6ZmFsc2UsXCJlbmFibGVfc3RlcF91cFwiOmZhbHNlfSxcIm5lbWlkXCI6e1wicHJpdmF0ZV90b19idXNpbmVzc1wiOmZhbHNlfX0iLCJhdWQiOiJodHRwczovL2xvY2FsaG9zdDo1MDAxIiwiZXhwIjoiMTY0NDQ4NTM1MiIsImlzcyI6Ijg3OWRkZDYyLTMxYzYtNDE1YS1hZjMyLWNkZWM0NzkxN2RkZCIsImlhdCI6IjE2NDQ0ODUwNTIiLCJuYmYiOiIxNjQ0NDg1MDUyIn0.RkLuLQEw6DyFhDBI7wM2EG1XIQ5RqFf3iYBfAEwcqYg
```

Discovery entry of encryption certificate:

```
{"kty":"RSA","use":"enc","kid":"9660F7AFFF397D67B77C92ED96F22A5A151F14C6","x5t":"9660F7AFFF397D67B77C92ED96F22A5A151F14C6","x5c":["MIIDGzCCAgOgAwIBAgIJAMbMrcpa3IbfMA0GCSqGSIb3DQEBCwUAMDQxMjAwBgNVBAMTKU5ldHMgZUlEIEJyb2tlciBjbGllbnQgcmVxdWVzdCBlbmNyeXB0aW9uMCAXDTIyMDIwNzEzMzcyNVoYDzIxMTIwMjA4MTMzNzI1WjA0MTIwMAYDVQQDEylOZXRzIGVJRCBCcm9rZXIgY2xpZW50IHJlcXVlc3QgZW5jcnlwdGlvbjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBALwXpFf5YpFNL3M1BEA4P8BJKmuoRe57vNDBLc\u002BekPWE2roH3I40z0X/uAlv6NVxULpQ3ILX8J4/G8xP8JUyrpAswI3ydT6R/iUDzswzzfNHwv4lp/ifbmLo\u002BQOXtvh\u002BYC2NcLHs54r9hugk0LjdLSaqGip84cMgYjE25PPoC/3aPtVN6I4hqPTK8GFqjRJjf5hlTaV1Ft3rZBqeYaTUYFimmytDbPz03I/Sv3RbiCI05PCVJlEituraECePJtXZYhTyFBjmJc0Yn33EO1skJ3wvPXXmpq2UJBNQMBkkZ0njza35NDbp2Y6QIkqbpsKQejh6ATPmyhbi8urjF94deI0CAwEAAaMuMCwwCwYDVR0PBAQDAgQQMB0GA1UdDgQWBBTS1x/H5ASUPiO51qufsHR6hC5IDjANBgkqhkiG9w0BAQsFAAOCAQEAuOAMuB4d81/O\u002BwAR8Ki52abP2jErdpE2NgZPRS\u002BxUsnJQjOtJc0FZYV54qwtYFokUEMFCaaNCVvimX5/1cJzmZ50cl78gZA0shL4WcKr2\u002BAOzbR6hmsvylYELMYefqQroskZFhbKM\u002BTvbbuTmiDtvEgj317WV2Z8Dyo4r11IBQKJE0\u002BOKz7Vvcg9b991Oq/JFNOyjNRr5yyZT9yIScTpZPKB5bQ8ZFE8lmn8hnHpx5bxBNGPTQ4dzLLTCGC4PkGmnwC/b55JFhRedpGJT9STgJ2aOCRUjjYHCTTFx8V9LnR6cqOpNDhtye83eX8RyIxIjYvJDja030QK2he5Xsinbw=="],"alg":"http://www.w3.org/2001/04/xmlenc#rsa-oaep"}
```
