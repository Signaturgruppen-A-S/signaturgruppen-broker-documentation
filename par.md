---
title: Pushed Authorization Request (PAR)
layout: home
nav_order: 11
parent: OpenID Connect integration
---

# Pushed Authorization Request (PAR)

The **Pushed Authorization Request (PAR)** is a specification in the OAuth 2.0 and OpenID Connect (OIDC) frameworks, designed to improve the security and reliability of authorization requests. Instead of sending authorization parameters directly to the authorization endpoint via the user's browser, PAR allows clients to securely push these parameters to the authorization server in a direct request.

## Key Features of PAR
1. **Enhanced Security**: By avoiding the transmission of sensitive data (e.g., `redirect_uri`, `scopes`) in the front channel, PAR mitigates risks such as query parameter tampering or leakage in browser history.
2. **Simplified Request Validation**: The authorization server can validate the parameters in advance, ensuring consistency and reducing errors at the authorization endpoint.
3. **Support for Complex Requests**: PAR facilitates passing larger or more complex request objects, particularly when using JSON Web Tokens (JWTs) or encrypted data.

---

## PAR Workflow

1. **Client Pushes Authorization Request**:  
   The client sends a `POST` request to the **PAR endpoint** with the full set of authorization parameters.
2. **Authorization Server Returns a Request URI**:  
   The server validates the request and responds with a unique `request_uri` (a reference to the pre-validated request).
3. **User Initiates Authorization**:  
   The client uses the `request_uri` in a standard authorization request sent to the authorization endpoint.

---

### Example PAR Flow

1. **Client Pushes Authorization Request**:
   ```http
   POST /as/par HTTP/1.1
   Host: authorization-server.example.com
   Content-Type: application/x-www-form-urlencoded
   Authorization: Basic <Base64(client_id:client_secret)>
   
   client_id=client123
   &response_type=code
   &redirect_uri=https%3A%2F%2Fclient.example.com%2Fcallback
   &scope=openid%20profile
   ```

2. **Authorization Server Responds**:
   ```http
   HTTP/1.1 201 Created
   Content-Type: application/json
   
   {
     "request_uri": "urn:example:request:abc123",
     "expires_in": 300
   }
   ```

3. **Client Redirects User to Authorization Endpoint**:
   ```http
   GET /authorize?client_id=client123&request_uri=urn:example:request:abc123 HTTP/1.1
   Host: authorization-server.example.com
   ```

4. **User Authorizes and Completes Flow**:  
   The server retrieves the pre-validated parameters from the `request_uri`, processes the request, and completes the authorization code grant or token issuance.

---

## Benefits of PAR
- **Stronger Security**: Prevents exposure of sensitive data in user-agent-facing requests.
- **Pre-Validation**: Catches errors early in the flow, improving developer experience.
- **Compliance**: Aligns with regulations and security best practices for transmitting sensitive authorization data.

---

## References
- [OAuth 2.0 Pushed Authorization Requests (RFC 9126)](https://www.rfc-editor.org/rfc/rfc9126)
- [OpenID Connect Core Specification](https://openid.net/specs/openid-connect-core-1_0.html)