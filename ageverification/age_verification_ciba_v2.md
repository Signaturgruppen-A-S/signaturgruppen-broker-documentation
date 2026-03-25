# Age Verification (V2)
**DISCLAIMER: This document is under construction and review and has not been finalized nor implemented yet.**

## Introduction to Age Verification (AV)
To support Age Verification (AV), Signaturgruppen Broker has implemented a streamlined and robust interface that allows for an easy, compliant and flexible integration from any system and adaptable to any workflow. 

Data minimization is primary - only the AV result is returned. 

The integration interface is minimal and stable, while the administrative UI allows for flexible customization for integrations. This supports stable integrations that is easily adaptable for changes over time, without the need to changes to the integrating service.

### EU AV and alternatives
The primary AV mechanism supported is the European Wallet supported AV flow, which is specifically tailored to be data minimalistic, secure, anonymous and widely available cross borders across the EU. This includes support for national AV solutions under the EU AV scheme, such as the danish AltID.

Other AV providers such as the danish MitID and e-Boks (non-exhaustive list) is supported and can be configured and enabled for the integration via the administrative UI.

## CIBA

Official docs: [https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html).

### Example CIBA request
The following example demonstrates a standard initialization of an AV flow using our CIBA interface.

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid av:18' \
--data-urlencode 'client_id=[your_client_id]' \
--data-urlencode 'client_secret=[your_secret]' \
--data-urlencode 'nonce=[your-nonce]'
```

> Note that you can use client assertion signed JWTs instead of posting client secret directly - any OIDC/CIBA supported mechanics for client secret is supported.

Response:
```
{
  "auth_req_id": "9384B..-1",
  "expires_in": 300,
  "interval": 3,
  "av_uri": "https://pp.idbroker.eu/op/connect/authorize?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2",
  "av_iframe_src": "https://pp.idbroker.eu/op/av/iframe?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2",
  "av_qr_png_b64": "iVBORw..."
}
```

### CIBA Poll, Ping, and Push Modes
For reference, see the [official documentation for response modes here](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html#rfc.section.5).

The client is default configured for poll mode, but can be set for ping or push mode in the admin UI. 



### Controlling last redirect for the end-user
The default behavior for the last redirect in the end-user browser flow is a landing page controlled by Signaturgruppen Broker, that informs the end-user of the flow result and informs that the page can be closed.

It is possible to control the last redirect of the end-user, by specifying a **redirectUri** parameter in the CIBA PAR initialization call:
```
{
    "requestId": "9384B4AA93780A1AB83684ACE8B07DFCB6C038E9A6847565DC4F0F2547A499D6-1",
    "redirectUri": "[your-https://yourdomain.site/return-after-av-flow]",
    "state": "[your-flowspecific-identifier]"
}
```

## Example ID token
A successful flow will result in an ID token, which contains the relevant claims issued for the flow.

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
   "transaction_id": "90..32",
   "idtoken_type": "av",
   "av:18": "true"
}
