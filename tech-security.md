---
title: Security
layout: home
nav_order: 20
---

# Security
This section will cover supported cryptographic algorithms, supported TLS versions, certificate pinning and other security related issues.
OpenID Connect provides a high level of security, but for some application require additional security hardening. This section will cover the available options.

## PKCE
Proof Key for Code Exchange by OAuth Public Clients (PKCE) is an extension to the Authorization Code flow to prevent certain attacks and to be able to securely perform the OAuth exchange from public clients.
This is prevalent and recommended when integrating from a mobile (Android and iOS) platform and allows the initiating app to be the only one who can retrieve the issued tokens, even though the client is a public client.
PKCE is fully supported. See [PKCE](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#pkce) for reference.

## Server certificate validation for TLS
When calling any of the Signaturgruppen Broker APIs or endpoints the integrity of the TLS certificate presented can be verified using the following checks

* That the host name indicated in the certificate matches the host name of the APIs URL
* That the presented certificate is issued by one of the publicly trusted CA’s listed in the documentation
* That the certificate is within its validity period
* That the signature in the certificate is valid

As an example, this can be used to verify the validity of the signing keys available from the Discovery endpoint.

## Signaturgruppen Broker signing keys
Unless otherwise specified or configured all signed tokens issued by Signaturgruppen Broker will be signed by an HSM protected key at compliance level supporting the [FIPS 140-2](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#fips1402) level 3 or equivalent.
All publicly available certificates are found through the OpenID Connect Discovery endpoint (TLS protected).
When signing tokens, Signaturgruppen Broker will use the algorithm ES256 (ECDSA using P-256 and SHA-256) or stronger.

### Pinning signing certificates
The signing certificates used for all signed tokens will only be changed if required. This would include if the signing key is somehow compromised.
There will be support for getting upcoming signing certificate change notifications by email or by other mechanism via a bilateral agreement with Signaturgruppen Broker. It is expected, that signing certificates will be changed only if required due to security concerns.
Unless otherwise specified, the token signing certificates for will be self-signed with a very long time-to-live (10+ years).

The signing certificate for transaction tokens will be an OCES3 organization certificate, issued by the NemLog-In3 OCES3 CA. The DN of the transaction token signing certificate is available in the Environment section in this document, allowing pinning of the certificate DN.
The signing certificates will be made available through our documentation and through the agreed communication channels, allowing pinning of the specific certificates.

## Supported HTTPS/TLS versions
All endpoints will require TLS 1.2 or higher.
The general guidelines and requirements for MitID Brokers will be adhered to, as a minimum and updated continuously.

## JWT, JWS and JWE tokens
JWS (JSON Web Signature) and JWE (JSON Web Encryption) are the signed and encryption versions of JWT (JSON Web Token).
Signaturgruppen Broker always uses the JWS/JWE compact serialization format.
Note, that JWT tokens are always represented as either JWS or JWE.

## Verification of tokens

### ID token and Userinfo token
The basic checks are often implemented by the OIDC client library used for the integration.
To verify the authentication response the following steps from the OIDC specification are validated: .

* The client MUST validate that the expected restrictions for amr, idp and identity_type are as expected.
* The client MUST validate that the expected restrictions for identity provider specific claims are as expected. Example could be MitID claims: loa, ial and aal.

### Transaction token

* The expected Issuer Identifier MUST exactly match the value of the iss (issuer) claim.
* The client MUST validate the signature of transaction tokens according to JWS [JWS](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#jws) using the algorithm specified in the JWT alg Header Parameter. The client MUST use the keys provided by the Issuer (available via the Discovery endpoint).
* The client MUST validate that the signing certificate is the certificate specified as the “Transaction token signing certificate” in the environments section in this document.
* The iat Claim can be used to reject tokens that were issued too far away from the current time, limiting the amount of time that nonces need to be stored to prevent attacks. The acceptable range is client specific.
* If the auth_time claim was requested, either through a specific request for this claim or by using the max_age parameter, the client SHOULD check the auth_time claim value and request re-authentication if it determines too much time has elapsed since the last end-user authentication.
* The client MUST validate that the expected restrictions for acr, ial, amr, idp and identity_type are as expected.
* Optionally, validate the OCSP response of the signing certificate, set as transaction_token_ocsp_resp in the token response (currently not implemented, will be released soon).
* Optionally, validate the NONCE parameter by correlating the transaction_id found in the transaction token with the ID token, as the NONCE parameter is only set in the ID token.

## Verification of UserInfo endpoint response
* Due to the possibility of token substitution attacks the UserInfo response is not guaranteed to be about the end-user identified by the sub (subject) element of the ID token. The sub claim in the UserInfo response MUST be verified to exactly match the sub claim in the ID token; if they do not match, the UserInfo response values MUST NOT be used.
* The client MAY introduce additional security measures by pinning the TLS certificates or by requesting a signed response.
