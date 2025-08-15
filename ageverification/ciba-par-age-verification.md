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

## User experience
The flow is initiated from the integrating service backend, and then continously polled until the resulting ID token with the verification response is fetched. 

At https://brokerdemo-pp.signaturgruppen.dk/ageverifyqr we have setup an interactive QR code demo of an Age Verification Flow, which utilizes this API under the hood. Here: 

1. CIBA + PAR is initiated
2. QR is created from resulting **authentication_uri** and show to the end-user
3. User is able to scan the QR to initiate a MitID (PP: https://pp.mitid.dk/test-tool/frontend/#/create-identity) age verication flow (or manually click the new tab link below the QR for demo purposes)
4. The QR page will poll in the background for status update (using a backend) - the new tab/QR opened browser on another device will start the OIDC PAR age verification flow.
5. When the flow is completed the end-user will see a success page and the QR page will update with the result of the flow.

**MitID age verification text:**  
![MitID age verification text](https://github.com/user-attachments/assets/8250ae01-b623-4342-a35a-8a7773119d9a)

**User browser completed:**  
![User browser completed](https://github.com/user-attachments/assets/9358bbfd-22f1-40cc-96aa-9c744006561d)

**QR page updated with status:**  
![QR page updated with status](https://github.com/user-attachments/assets/ee79314b-e719-492b-8244-9e0f68a4397f)


