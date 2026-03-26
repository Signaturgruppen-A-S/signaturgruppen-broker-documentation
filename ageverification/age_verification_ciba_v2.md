# Age Verification (V2)
**DISCLAIMER: This document is under construction and review and has not been finalized nor implemented yet.**

## Introduction to Age Verification (AV)
To support Age Verification (AV), Signaturgruppen Broker has implemented a streamlined and robust interface that allows for an easy, compliant and flexible integration from any system and adaptable to any workflow. 

Data minimization is primary - only the AV result is returned. 

The integration interface is minimal and stable, while the administrative UI allows for flexible customization for integrations. This supports stable integrations that is easily adaptable for changes over time, without the need to changes to the integrating service.

### EU AV and alternatives
The primary AV mechanism supported is the European Wallet supported AV flow, which is specifically tailored to be data minimalistic, secure, anonymous and widely available cross borders across the EU. This includes support for national AV solutions under the EU AV scheme, such as the danish AltID.

Other AV providers such as the danish MitID and e-Boks (non-exhaustive list) is supported and can be configured and enabled for the integration via the administrative UI.

## Supported AV integration variants
Signaturgruppen Broker supports two primary integration protocols for AV integration, namely CIBA and a browser-based iframe variant. 
These two variants cater to different integration requirements and needs and provides support for most integration scenarios. 

The CIBA protocol support a backend initiated well documented workflow with flexible options and ways to receive the result directly to your backend.
The iframe protocol supports a browser initiated integration that allow for easy and flexibile integration into existing web-based setups that have a harder time to adjust their workflows using the CIBA protocol.

## CIBA
Official docs: [https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html).

The CIBA integration support a backend to backend protocol, with possibility of setting up both

### Example CIBA request
The following example demonstrates a standard initialization of an AV flow using our CIBA interface.

```
curl --location 'https://pp.idbroker.eu/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid av:18' \
--data-urlencode 'client_id=[your-client-id]' \
--data-urlencode 'client_secret=[your-secret]' \
--data-urlencode 'reference_id=[your-reference-id]'
```

> Note that you can use client assertion signed JWTs instead of posting client secret directly - any OIDC/CIBA supported mechanics for client secret is supported.

Example response (poll mode):
```
{
  "auth_req_id": "9384B..-1",
  "expires_in": 300,
  "interval": 3,
  "av_uri": "https://pp.idbroker.eu/op/connect/authorize?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2"
}
```
### Age scope(s) (av:age)
The requested age, age or older, is requested via the scope parameter **age:[age]**. Requesting credential proving that the user is 18 or older is done with the **age:18** scope value.

Multiple age verification credentials can be requested simultaneously, such as setting both **age:16** and **age:18** in the scope value. This will result both credentials in the result.

### Optional init parameters

| Name | Description |
|--------|--------|
| client_notification_token        | Required for ping and push modes  |
| redirect_uri       | (Optional) Specifies a redirect for the end-user when the flow has completed. Not all AV flows support this, as some wallets might not support redirecting the end-user when done. |
| redirect_state       | (Optional) When a redirect_uri parameter is specified and the end-user is redirected, the redirect_state will be appended as query/post parameter       |
| redirect_type       | (Optional) Specifies redirect type. Values: get (default) and post.   |


ping/push bearer
--data-urlencode 'redirect_uri=[your-redirect-state]' \
--data-urlencode 'redirect_state=[your-redirect-state]' \
extra age scopes
client_notification_token
-- iframe ciba vs iframe browser

### Optional init response values (Admin UI)
  "av_iframe_src": "https://pp.idbroker.eu/op/av/iframe?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2",
  "av_qr_png_b64": "iVBORw..."

### CIBA Poll, Ping, and Push Modes
For reference, see the [official documentation for response modes here](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.5).

The client is default configured for poll mode, but can be set for ping or push mode in the admin UI. Please reach out if you are interested in the ping or push mode.

### Example ID token (CIBA)
A successful flow will result in an ID token, which contains the relevant claims issued for the flow.

```
{
   "iss": "https://pp.idbroker.eu/op",
   "nbf": 1725009225,
   "iat": 1725009225,
   "exp": 1725009525,
   "aud": "[your client id]",
   "reference_id": "[your-reference-id]",
   "sub": "[random sub]",
   "auth_time": 1725009225,
   "idp": "eu_av",
   "transaction_id": "90..32",
   "idtoken_type": "av",
   "av:18": "true"
}
```

## Iframe integration
Documentation under development.
