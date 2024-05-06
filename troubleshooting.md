---
title: Troubleshooting
layout: home
nav_order: 60
---

# Introduction

This document aims at giving service providers an overview of the most general and basic error cases experienced when integrating to the Nets eID Broker system.

The intended audience is IT developers.

General information, online demonstration, documentation (including the newest version of this document) and example code is found at <https://broker.signaturgruppen.dk>.

Ensure that you have a basic understanding of the OIDC flow. In this document we will walk through the most common caveats in the relevant steps of the overall protocol:

1. Initiating the Flow
2. Accessing tokens
3. Calling Userinfo or other endpoints
4. Performing browser-based logout

# Basic integration

The most basic integration for a service provider in Open ID Connect is based on a so-called client. A client is set up in the administration portal, <https://pp.netseidbroker.dk/admin> (pp) and login is based on MitID PP test-identities. Contact us if you require access to the portal.

A customer is first set up as an organization, based on name adn VAT-registration number. Within the organization one or more service providers can be configured. A service provider can hold a list of configured Identity Providers, like MitID and a list of services.

A service is what controls which UX should be shown with MitID, further a service may hold a list of individual Open ID Connect clients. Each client can be configured with various parameters as detailed in the table below

|     | **Description** | **Examples** |
| --- | --- | --- |
| **General** | Basic client-information | Client ID 80e5e4c4-d8f1-424f-8411-589caf1c05cd<br><br>Client Secret DcOwmQBDGy/L85so1X/onNDLPEUNCENJAl2rY5nt7k4HzztXYrUKKgk1jaY51JB6iir4FuMbz3prtZYHrq5SWQ== |
| **Endpoints** | Whitelisting of URLs to be used in the flow. Note that the PP environment supports allowing all URLs | Login Redirect <https://www.customer.dk/sign-inRedirect> Origin <https://www.cutomer.dk>  <br>Logout Redirect <https://www.customer.dk/logged-out>  <br>App Switch Url <https://app.customer.dk/app-switch> |
| **IdP** | List of available IdP to be used within the client | MitID and MitID Erhverv are examples. Note that an IdP can be available, which is required if it should be used in a flow. If you do not wish to control it on runtime through the idp_values parameter you should also enable the IdP by default.  <br>Note that if more than one IdP is enabled in a flow an IdP-selection page is shown |
| **Advanced** | Various more advanced configuration options | Grant type Code |

# Initiating the flow

When the Service Provider initiates the MitID flow a URL is generated and the user is sent to this address. The URL is built by the OpenID connect implementation and based on the  [discovery endpoint (PP)](https://pp.netseidbroker.dk/op/.well-known/openid-configuration) of the broker. 

The URL used for the OpenID Connect authentication request (redirect), could look something like this:
```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=80e5e4c4-d8f1-424f-8411-589caf1c05cd&redirect_uri=https://openidconnect.net/callback&scope=openid%20mitid&response_type=code&idp_values=mitid_erhverv
```
Note that the client id is identified for the broker to identify which service and service provider the user should be shown. The scope is identified in order to indicate which information is expected back after the authentication is completed. The OpenID Connect grant type is indicated in the response_type.

## Possible error states

The table below shows some normal causes for errors observed when redirecting the user the initial login, the authorize-endpoint.

| **Error message** | **Cause** | **Remediation** |
| --- | --- | --- |
| unauthorized_client<br><br>Unknown client or client not enabled | Wrong client ID used | Ensure that the client id is specified precisely as it is in the admin-portal or received from support. Note that a client id in production does not work in PP or vice versa. |
| invalid_request<br><br>Invalid redirect_uri | Wrong redirect URL used | Ensure that the redirect URL used is whitelisted in the admin-portal. |
| unsupported_response_type<br><br>Response type not supported | Wrong response type used | Ensure that the response type used matches the configuration in the admin-portal. |
| invalid_request<br><br>Invalid nonce | Missing Nonce | Note that when implicit or hybrid flow type is used nonce is a requirement. |
| invalid_request<br><br>Requested IDP is not enabled for this client | The idp_values runtime parameter requests an idp which is not available for this client | Ensure that the idp_values parameter used matches the configuration in the admin-portal |

# Accessing tokens

After the user has completed the flow he is returned to the service provider typically with a code. This code can be exchanged to tokens at the token-endpoint. Open ID Connect implementations typically localize the token endpoint from the discovery endpoint. The payload is a HTTP post towards <https://pp.netseidbroker.dk/op/connect/token>

The following parameters are posted

```
POST https://pp.netseidbroker.dk/op/connect/token

grant_type: "authorization_code"
redirect_uri: "<https://openidconnect.net/callback>"
code: "53261AB9D54E62E01A84C0AA8DF4812CD8BC9420B577D72E3743DE5C830EB953-1"
client_id: "80e5e4c4-d8f1-424f-8411-589caf1c05cd"  
client_secret: "DcOwmQBDGy/L85so1X/onNDLPEUNCENJAl2rY5nt7k4HzztXYrUKKgk1jaY51JB6iir4FuMbz3prtZYHrq5SWQ=="
```

## Possible error states

The table below shows some normal causes for errors observed when accessing the token endpoint.

| **Error message** | **Cause** | **Remediation** |
| --- | --- | --- |
| "error": "invalid_grant" | Wrong code or redirect URL is used | Ensure that the provided code is not used before and that the redirect URL used is exactly the same as when initiating the flow. |
| "error": "invalid_client" | Wrong client used | Ensure that the correct client ID and secret is used. |

# Calling Userinfo or other endpoints

After fetching the tokens from the token endpoint the normal next step is to call eg the userinfo endpoint in order to get access to user information. Calling is based on using the received access token as Bearer token. The userinfo endpoint can again be localized from the discovery endpoint and it is available at 

```
https://pp.netseidbroker.dk/op/connect/userinfo
```

## Possible error states

The table below shows some normal causes for errors observed when accessing the userinfo endpoint.

| **Error message** | **Cause** | **Remediation** |
| --- | --- | --- |
| Missing information in userinfo | Wrong scope used | Ensure that the scope parameter used when starting the flow matches your requirements, i.e. it should be set to include mitid in order for userinfo to hold information on private MitID users and similarly nemlogin for users MitID Erhverv users. |
| "error": "invalid_client" | Wrong client used | Ensure that the correct client ID and secret is used. |

Note especially that missing CPR is most often due to not having ssn inicluded in scope when starting the session.

# Performing browser-based logout

User sessions can be manipulated in various ways. See the **\[NEB-SESSIONS\]** for details. If you wish to log-out the user from his MitID/Broker-session through the browser this is done with a redirect to a URL available at the discovery endpoint. The URL looks something like this  

```
https://pp.netseidbroker.dk/op/connect/endsession?post_logout_redirect_uri=https://brokerdemo-pp.signaturgruppen.dk/signout-callback-oidc&id_token_hint=eyJhbGciOiJSUzI1NiIsImtpZCI6>...
```

## Possible error states

The table below shows some normal causes for errors observed when accessing the logout endpoint.

| **Error state** | **Cause** | **Remediation** |
| --- | --- | --- |
| User is not redirected back to specified URL | Wrong parameters used | Ensure that the id_token specified in the URL is still valid and that the given post_logout_redirect URL parameter specified is indeed whitelisted in the admin-portal for the client. |

# Special cases

This section holds various tips on integration details.

## The use of user specific identifiers/invitation links

One specific scenario is for users to be invited using an invitation id passed on in the URL as one of the query parameters. That may be a challenge with MitID flows and special care must be taken for the invite query parameter to be preserved in the OIDC flow in order to secure that the user is redirected to a specific page after successfully signing-up.

Here are some strategies you can use to handle this scenario:

1. **State Parameter**: The OpenID Connect protocol includes the **state** parameter, which is intended to maintain state between the authentication request and the callback from the authorization server. This parameter is returned to your callback URL unchanged and is useful for correlating the request with the response or for storing user states like the original URL. You can store your invitation ID in the **state** parameter.
2. **Use a Persistent Cookie**: Set a cookie with the invitation ID before redirecting the user to the authentication provider (MitID/NEB). Once the authentication process completes and the user is redirected back to your application, you can retrieve the invitation ID from the cookie.
3. **Session Management**: Store the invitation ID in the server-side session before redirecting the user. After the user is authenticated and redirected back to your application, retrieve the invitation ID from the session.
4. **Customize the Redirect URI**: If you sign your OpenID Connect requests, Signaturgruppen Broker allows dynamic **redirect_uri**'s, which enables setting the id or invitation link directly in the **redirect_uri**.

Given your specific integration with MitID, the most robust and standard approach would involve using the **state** parameter. Here’s how you can implement it:

- When initiating the OIDC flow, store the invitation ID in the **state** parameter.
- Encode (and possibly encrypt, for added security) the **state** value to include the invitation ID.
- After the user is authenticated and redirected back, decode the **state** parameter to extract the invitation ID.

This approach ensures that the invitation ID is preserved without relying on browser cookies or session, and aligns with the OpenID Connect specifications which are designed to handle scenarios like this securely.
