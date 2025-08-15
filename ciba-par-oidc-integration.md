---
title: CIBA PAR OIDC integration
layout: home
nav_order: 22
---


# CIBA + PAR integration
Signaturgruppen Broker supports a special [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) integration, that allows a service provider to initate any flow normally available via Signaturgruppen Brokers OpenID Connect interface via CIBA.

This allows a purely backend initiated flow in which a backend service is able to initiate a flow and continuously poll for flow completion and upon completion receive the normal Token ednpoint result (OIDC tokens).
Signaturgruppen Broker has implemented this by introducing a special API endpoint that allows a service to convert an initiated CIBA flow into a PAR authentication URL which can be opened in any supported browser in which the end-user will complete the requested flow.

This can help various setups to integrate flows into a range of integrations with better compatability across applications such as mobile apps, web applications and backend services. This also allows for setting up flows where the end-user completes the flow on a different device and/or browser than from where the flow is initiated, allowing for better flexibility and support for various flow types. 

# CIBA integration
Signaturgruppen Broker supports CIBA integration for various flows and follows the [CIBA spec](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) with the CIBA endpoint reference found via the discovery endpoint, see environment section for reference to the [authority URL](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/environments.html) of Signaturgruppen Broker.

## CIBA parameters
The **scope**, **client_..** and **grant_type** parameters is set as a top-level CIBA request parameters, with the same effect as for the corresponding OIDC flow.
The **login_hint_token** parameter is required for the CIBA initialization invocation. 
Any other top-level parameters normally set for Signaturgruppen Broker OIDC integrations is set as part of the **login_hint_token** object.

### **login_hint_token** structure
Following a non-exhaustive table of options:
| Parameter | Values | Required | Description |
|----------|----------|----------|----------|
| flow_type    | broker_oidc   | Required   | Must be set to the value **broker_oidc**   |
| idp_values    | Single space-delimited string | Optional   | Example: "mitid" or "mitid mitid_erhverv"   |

The **login_hint_token** must be constructed as a valid JSON structure, setting each of the required and optional values as JSON key-value pairs.
For example: 
```
login_hint_token={"flow_type": "broker_oidc" }
```

## Example MitID flow
The following examples demonstrate how to setup a MitID login flow using the CIBA PAR mechanics, setting up appropriate values for the flow.

- Initiate CIBA flow, retrieve **request_id**
- Initiate OIDC PAR flow using **request_id**
- Initiate flow in browser: Open browser with url, app switch from mobile app, generate QR code for users to scan, ...
- Poll result (pending, error, success)

### Initiate CIBA flow, retrieve **request_id**:
First a CIBA initialization request is made:
```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid mitid' \
--data-urlencode 'client_id=b....7' \
--data-urlencode 'client_secret=a....SuaQ==' \
--data-urlencode 'login_hint_token={"flow_type": "broker_oidc" }' \
--data-urlencode 'prompt=login' \
--data-urlencode 'idp_values=mitid'
```
Then a CIBA request ID is received:
Response:
```
{
    "auth_req_id": "9384B4AA93780A1AB83684ACE8B07DFCB6C038E9A6847565DC4F0F2547A499D6-1",
    "expires_in": 300,
    "interval": 3
}
```

### Initiate OIDC PAR flow using **request_id**
To initiate a PAR OIDC flow based on the initiated CIBA **request_id**, the CIBA PAR init auth endpoint is invoked:
```
{
    "requestId" : "9384B4AA93780A1AB83684ACE8B07DFCB6C038E9A6847565DC4F0F2547A499D6-1"
}
```

Giving a response on the form: 
```
{
    "request_uri": "urn:ietf:params:oauth:request_uri:4BEE3FBF81FBD52F997575BDBBC9459881E924E6535433D0427A7E46E8D7DB52",
    "expires_in": 600,
    "authentication_uri": "https://pp.netseidbroker.dk/op/connect/authorize?client_id=b....7&request_uri=urn:ietf:params:oauth:request_uri:4BEE3FBF81FBD52F997575BDBBC9459881E924E6535433D0427A7E46E8D7DB52"
}
```

### Initiate flow in browser
To start the flow for the end-user, the returned **authentication_uri** must be opened in the end-users browser. This can be on the same device or on a different device and it is up to the integrating service to transfer this URL and to open the browser for the end-user.
As an example, the service could render a QR code of the **authentication_uri** which would allow any user to scan it with their mobile camera app and automatically start the flow in their default browser.

### Poll result
Then the backend uses the CIBA poll mechanism, via the OIDC Token endpoint, to continously poll for status update for the initiated flow. The special CIBA pending result indicates that the flow is still ongoing, all other responses indicate either some error or success and will only be returned once, then the **invalid_grant** will be returned.

Pending:
```
{
    "error": "authorization_pending"
}
```
Error:
```
{
    "error": "[some_error]"
}
```
Success:
```
{
    "id_token": "eyJh..NLjg",
    "access_token": "ey..cI5fHRKnQ",
    "expires_in": 10800,
    "token_type": "Bearer",
    "scope": "openid mitid"
}
```

## Controlling last redirect for the end-user
The default behavior for the last redirect in the end-user browser flow is a landing page controlled by Signaturgrupen Broker, that informs the end-user of the flow result and informs that the page can be closed.

It is possible to control the last redirect of the end-user, by specifing a **redirectUri** parameter in the CIBA PAR intitialization call:
```
{
    "requestId" : "9384B4AA93780A1AB83684ACE8B07DFCB6C038E9A6847565DC4F0F2547A499D6-1",
    "redirectUri" : "https://yourdomain.dk/cibaparflowcompleted",
    "state": "your-flowspecific-identifier"
}
```

This redirect will receive the optional **state** parameter for the CIBA PAR initialization call as query parameter, allowing for more tailored handling of the last step of the protocol.
Error flows will also be sent to the set **redirectUri** applying an additional **error** and **error_description** query parameter.

