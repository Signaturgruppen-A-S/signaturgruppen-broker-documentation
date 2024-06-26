---
title: API integration
layout: home
nav_order: 11
---

# API integration
Signaturgruppen Broker and supporting services utilizes to primary ways to authorize access to published APIs
* Using end-user access tokens (openid connect)
* Using services tokens (client credentials)

This section will focus on the basics for utilizing an API client for client credentials flow.

## Getting an API client
The Signaturgruppen Broker admin interface allows for the creation of clients for either OpenID Connect or client credentials (API clients) integration.

If your organization have access to the admin interface, then you can create your own clients. 
You are also welcome to write to Signaturgruppen support an request the required API clients.

### API clients
An API client is a OAuth client consisting of
* Client ID
* Client Secret

Is is possible to both setup standard shared secrets or upload a public key for a RSA public/private keypair.

## OAuth2 Client Credentials Flow: Technical Description
### Overview
The OAuth2 Client Credentials Flow is a method for an application to authenticate itself with an authorization server to gain access to protected resources. This flow is primarily used for machine-to-machine (M2M) communication, where there is no user interaction. The application uses its own credentials to obtain an access token.

### Steps of the Flow

1. **Client Authentication**:

* The client (application) authenticates itself to the authorization server using its client ID and client secret.

3. **Token Request**:

* The client sends a POST request to the token endpoint of the authorization server, including its client ID, client secret, and the grant type.

3. **Token Response**:

* If the client credentials are valid, the authorization server responds with an access token.

4. **Accessing Resources**:

* The client uses the access token to make authorized requests to the protected resources on the resource server.

### Detailed Flow

1. **Client Authentication**:

* The client application has been registered with the authorization server and has received a client_id and client_secret.

2. **Token Request**:

* The client sends a POST request to the authorization server's token endpoint.
* Example:

```
POST /oauth2/token HTTP/1.1
Host: authorization-server.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=YOUR_CLIENT_ID
&client_secret=YOUR_CLIENT_SECRET
&scope=DESIRED_SCOPE
```

3. **Token Response:**

* The authorization server verifies the client's credentials.
* If valid, the server responds with an access token.
* Example:

```
{
  "access_token": "2YotnFZFEjr1zCsicMWpAA",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

4. **Accessing Resources:**

* The client uses the access token to access the protected resource.
* Example request:

```
GET /resource HTTP/1.1
Host: api.resource-server.com
Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA
```

* Example response:

```
{
  "data": "Here is the protected data."
}
```

### Example Scenario
**Setup:**
* Client ID: abc123
* Client Secret: secret

Token Request:

```
POST /oauth2/token HTTP/1.1
Host: authorization-server.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=abc123&client_secret=secret
```

Token Response:

```
{
  "access_token": "mF_9.B5f-4.1JqM",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Accessing Protected Resource:
```
GET /data HTTP/1.1
Host: api.resource-server.com
Authorization: Bearer mF_9.B5f-4.1JqM
```

Resource Server Response:

```
{
  "data": "This is the protected data."
}
```

### Key Points
* Client ID and Client Secret: These are used to authenticate the client application with the authorization server.
* Token Endpoint: The URL where the client application sends its request for an access token.
* Access Token: A token issued by the authorization server that the client can use to access protected resources.
* Grant Type: For this flow, the grant type is always **client_credentials**.
