---
title: Age Verification for Windows applications 
layout: home
parent: Age Verification
nav_order: 1
---

# Windows applications 
This section describes how to implement MitID Age Verification from a native Windows application.

# Implementation overview

1. **Use the System Browser**: Use the system’s default browser for authentication instead of embedded web views. This leverages the browser’s security features, reduces the risk of phishing attacks, and ensures users interact with a trusted interface.
Leverage Loopback Redirects: For native applications, use a loopback address (e.g., http://127.0.0.1) as the redirect URI. This minimizes potential interception of tokens and aligns with the recommendations in [RFC 8252](https://www.rfc-editor.org/rfc/rfc8252).

2. **Use the Authorization Code Flow with PKCE**
Authorization Code Flow: This is the recommended flow for native applications since it separates the authentication process from the token issuance.
Proof Key for Code Exchange (PKCE): Implement PKCE to add an extra layer of security by mitigating interception risks during the authorization code exchange process.
> Note, that this is a recommended security best practise for OAuth implicit public client flows, but for Age Verification flows it is an optional feature as the risk associated with interception of the ID token of a malign party is minimal. But you should as a service provider always do your own risk assesment of this.

4. **ID token validation (Required)**
Validate the received ID token: Always verify the ID token’s signature, check the token’s issued time, issuer, and audience to ensure it is valid and hasn’t been tampered with.
Implement Nonce and State Parameters: Use a nonce parameter in the authentication request to protect against replay attacks and a state parameter to mitigate CSRF risks.
The ID token received from the Age Verification flow is data minimized and does only contain the age verification information, no personal information is relayed in this token.
Expiration of the token is not important for age verification flows, it is more important to utilize the **nonce** parameter in order to ensure that the Age Verification result matches the correct request.

6. **Use Up-to-date and Well-supported Libraries**
Library Choices: Consider using libraries like MSAL (Microsoft Authentication Library) or IdentityModel.OidcClient which are designed for native applications and handle much of the complexity around OIDC flows.

7. **Follow OAuth 2.0 for Native Apps Guidance**
Security Recommendations: Align your implementation with the OAuth 2.0 for Native Apps best practices, which emphasize using system browsers, loopback interfaces, and secure token exchange methods.
HTTPS Everywhere: Ensure all endpoints, including redirect URIs and token endpoints, are secured via HTTPS to protect data in transit.
