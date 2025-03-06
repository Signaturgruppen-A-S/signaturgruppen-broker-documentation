---
title: Transaction token
layout: home
parent: Issued claims and tokens
nav_order: 1
---

# Transaction token

## Introduction

The transaction token will be available as a sealed (signed by a Nets MitID Broker OCES3 organization certificate) record of the end-user completed transaction.

The transaction token response is self-contained and cryptographically sealed record suitable for long-term storage and as a proof-of-transaction.

The transaction token includes the relevant authentication information, a unique transaction identifier as well as relevant transaction specific information like end-user approved text linked to the transaction.

## Transaction token

> **The transaction token will contain additional claims based on the end-user identity provider and provided identity provider parameters. These additional claims are explained under the specific identity provider in this document.**

Transaction tokens are meant as a receipt for the completed end-user transaction. The token is signed by a special signing key and formatted to support long-term verification of the end-user transaction.

Many of the issued claims, are the same as found in the accompanied ID token.

In this section all the claims that are always present in transaction tokens are specified. In addition, each identity provider will have its own set of claims that can be included depending the context.

Transaction tokens will be set in the Token endpoint response, if configured for the client and if requested using the **transaction_token** scope.

An accompanying OCSP revocation check response for the signing certificate, will be set in the Token endpoint response, formatted as a UTF-8+Base64 encoded string. The **signing_cert_ocsp_nonce** claim set in the transaction token is the nonce used for the OCSP response.

Token endpoint response:

```
{
  "id_token": "AGGSSDDhY3Rpb…AFGGRh",
  …,
  "transaction_token": "VHJhbnNhY3Rpb…BEYXRh",
  "transaction_token_ocsp_resp": "VHJhbnNhY3Rpb2…UZXN0IERhdGE="
}
```


Transaction tokens issued always include the following claims:

<table><tbody><tr><th><p><strong>Claim</strong></p></th><th><p><strong>Value</strong></p></th></tr><tr><td><p><strong>iss</strong></p></td><td><p>Identifier for the issuer as an URL using https scheme.</p></td></tr><tr><td><p><strong>sub</strong></p></td><td><p>NMB specific UUID representing the authenticated end-user.</p><p>Primary end-user identifier.</p></td></tr><tr><td><p><strong>iat</strong></p></td><td><p>Time at which the JWT was issued. Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.</p></td></tr><tr><td><p><strong>auth_time</strong></p></td><td><p>Time when the end-user authentication occurred.</p><p>Its value is a JSON number representing the number of seconds from 1970-01-01T0:0:0Z as measured in UTC until the date/time.</p></td></tr><tr><td><p><strong>nonce</strong></p></td><td><p>String value used to associate a client session with a transaction token, and to mitigate replay attacks. The value is passed through unmodified from the authentication request to the transaction token.</p><p>This is the same nonce as set in the ID token.</p></td></tr><tr><td><p><strong>signing_cert_ocsp_nonce</strong></p></td><td><p>Nonce for signing certificate OCSP revocation status check.</p><p>UTF-8+Base64 encoded string.</p><p>Nonce used for checking OCSP revocation status for the transaction token signing certificate after the token has been signed.</p></td></tr><tr><td><p><strong>amr</strong></p></td><td><p>Authentication method reference</p></td></tr><tr><td><p><strong>acr</strong></p></td><td><p>Authentication Context Class Reference.</p><p>Represents the Authenticated Level of Assurance (LoA). The [NSIS] framework forms the basis for how NMB handles LoA for authentications, but the available <strong>acr</strong> values are not restricted to the values specified in [NSIS].</p><p>Refer to the specific identity providers for a list of possible <strong>acr</strong> values.</p></td></tr><tr><td><p><strong>ial</strong></p></td><td><p>Identity Assurance Level</p><p>Strength of the Identity registration process.</p><p>Identity Assurance Level can be higher than the acr/LoA value, as the identity can be enrolled/registered at a higher level, than the achieved authentication level for a specific session.</p><p>If the ial is not known, this value is not set.</p></td></tr><tr><td><p><strong>idp</strong></p></td><td><p>Identity provider who issued the underlying identity.</p><p>Refer to the specific identity providers for a list of possible <strong>idp</strong> values.</p></td></tr><tr><td><p><strong>identitytype</strong></p></td><td><p>One of</p><ul><li>private</li><li>professional</li><li>test</li></ul></td></tr><tr><td><p><strong>transaction_id</strong></p></td><td><p>Transaction ID.</p><p>Unique identifier for the completed (end-user) transaction. Distinct identifiers are used to identity different transactions completed at NMB.</p></td></tr><tr><td><p><strong>recipient_info</strong></p></td><td><p>Type: JSON.</p><p>The top-level members of the <strong>recipient_info</strong> JSON object are:</p><ul><li><strong>organization.number</strong></li><li><strong>organization.name</strong></li><li><strong>organization.country</strong></li><li><strong>redirect_uri</strong>: The URL that the end-user is redirect to from NMB upon successful completing the transaction.</li></ul><p>The <strong>recipient_info</strong> parameter value is represented as UTF-8 encoded JSON.</p></td></tr><tr><td><p><strong>spec_ver</strong></p></td><td><p>0.9 for this version. (Still under development)</p></td></tr></tbody></table>

The receiving party should store both the transaction token and the OCSP response. Optionally, the signing certificate can be stored.

## Verification of transaction token

- The expected Issuer Identifier MUST exactly match the value of the iss (issuer) claim.
- The client MUST validate the signature of transaction tokens according to JWS \[JWS\] using the algorithm specified in the JWT **alg** Header Parameter. The client MUST use the keys provided by the Issuer (available via the Discovery endpoint).
- The client MUST validate that the signing certificate is a Voces3 certificate with the exact certificate DN specified in the environments section in this document.
- The iat Claim can be used to reject tokens that were issued too far away from the current time, limiting the amount of time that nonces need to be stored to prevent attacks. The acceptable range is client specific.
- If a nonce value was sent in the authentication request, a nonce claim MUST be present, and its value checked to verify that it is the same value as the one that was sent in the authentication request. The client SHOULD check the nonce value for replay attacks. The precise method for detecting replay attacks is client specific.
- If the **auth_time** claim was requested, either through a specific request for this claim or by using the **max_age** parameter, the client SHOULD check the **auth_time** claim value and request re-authentication if it determines too much time has elapsed since the last end-user authentication.
- The client MUST validate that the OCSP response validates for the signing certificate and that the OCSP response time is after the transaction token issued time (iat).
- The client MUST validate that the expected restrictions for **acr, ial**, **amr**, **idp** and **identitytype** are as expected.

## Transaction token MitID specific claims

| **Claim value** | **Possible values** |
| --- | --- |
| mitid.uuid | Same value as for ID token. |
| mitid.referencetext | Passthrough of the MitID **referencetext** identity provider parameter. |
| mitid.transactiontext | Passthrough of the MitID **transactiontext** identity provider parameter. |
| dk.cpr | Danish CPR.<br><br>Included if and only if configured as a requirement for the transaction token for the client in the administration interface.<br><br>Will trigger the same user interaction as setting the **ssn** scope. |
