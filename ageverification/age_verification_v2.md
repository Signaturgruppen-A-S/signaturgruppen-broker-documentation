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

### Example CIBA request
The following example demonstrates a standard initialization of an AV flow using our CIBA interface.

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid av:18' \
--data-urlencode 'client_id=[your_client_id]' \
--data-urlencode 'client_secret=[your_secret]' \
--data-urlencode 'login_hint_token={"nonce":"[unique-number-once]", "prompt": "login" }'
```

> Note that you can use client assertion signed JWTs instead of posting client secret directly - any OIDC/CIBA supported mechanics for client secret is supported.

Response:
```
{
  "auth_req_id": "9384B..-1",
  "expires_in": 300,
  "interval": 3,
  "av_uri": "https://pp.idbroker.eu/op/connect/authorize?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2",
  "av_qr_png": "[PNG]",
  "av_qr_svg": "[SVG]"
  "av_iframe_src": "https://pp.idbroker.eu/op/av/iframe?client_id=b.7&request_uri=urn:ietf:params:oauth:request_uri:4BEE..2"
}
```

