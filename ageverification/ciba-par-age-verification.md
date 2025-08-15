---
title: CIBA PAR Age Verification
layout: home
parent: Age Verification
nav_order: 5
---

# CIBA PAR Age Verification
Signaturgruppen Broker has a generalized [CIBA PAR OIDC](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/ciba-par-oidc-integration.html) mechanism that is relevant for age verification flows. 
Here an example of a standard "minimal age verification flow" is shown, which utilizes the **minimal** scope in order to force a minimalistic data-return, that helps adhere to GDPR and data-minimizing principles. 

## Age verification with minimal scope

In the following request, either **age** or **age_verify:[age]** can be used (not both):

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid minimal age' \
--data-urlencode 'client_id=[your_client_id]' \
--data-urlencode 'client_secret=[your_secret]' \
--data-urlencode 'login_hint_token={"flow_type": "broker_oidc" }'
```

> Note that you can use client assertion signed JWTs instead of posting client secret directly.

In the resulting ID token, either the **idbrokerdk_age=[age]** (using scope age) claim or the **idbrokerdk_age_verified=[age:bool(verified)]"** (using scope age_verify:[age]) is returned.
