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

In the following request, either **age** or **age_verify:[age]** scope can be used (not both):

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid minimal age' \
--data-urlencode 'client_id=[your_client_id]' \
--data-urlencode 'client_secret=[your_secret]' \
--data-urlencode 'login_hint_token={"flow_type": "broker_oidc", "idp_values":"mitid" }'
```

> Note that you can use client assertion signed JWTs instead of posting client secret directly.

In the resulting ID token, either the **idbrokerdk_age=[age]** (using scope age) claim or the **idbrokerdk_age_verified=[age:bool(verified)]"** (using scope age_verify:[age]) is returned.

## User experience
The flow is initiated from the integrating service backend, and then continously polled until the resulting ID token with the verification response is fetched. 

At [https://brokerdemo-pp.signaturgruppen.dk/ageverifyqr](https://brokerdemo-pp.signaturgruppen.dk/ageverifyqr) we have set up an interactive QR code demo of an Age Verification Flow, which utilizes this API under the hood. Here: 

1. CIBA + PAR is initiated
2. QR is created from resulting **authentication_uri** and show to the end-user
3. User is able to scan the QR to initiate a MitID (Use MitID PP Test-tool for simulation) age verication flow.
4. The QR page will poll in the background for status update (using a backend) - the new tab/QR opened browser on another device will start the OIDC PAR age verification flow.
5. When the flow is completed the end-user will see a confirmation page and the QR page will update with the result of the flow.

**MitID age verification text:**  
![MitID age verification text](https://github.com/user-attachments/assets/8250ae01-b623-4342-a35a-8a7773119d9a)

**User browser completed:**  
![User browser completed](https://github.com/user-attachments/assets/9358bbfd-22f1-40cc-96aa-9c744006561d)

**QR page updated with status:**  
![QR page updated with status](https://github.com/user-attachments/assets/ee79314b-e719-492b-8244-9e0f68a4397f)


## Sessions
If the user already has an active session with the client, accessing the **authentication_uri** will automatically redirect them to the confirmation page to complete the flow.

To prevent this behavior and require users to log in each time, set the **prompt** parameter in the **login_hint_token** to **login**:

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid minimal age' \
--data-urlencode 'client_id=[your_client_id]' \
--data-urlencode 'client_secret=[your_secret]' \
--data-urlencode 'login_hint_token={"flow_type": "broker_oidc", "idp_values": "mitid", "prompt": "login" }'
```

## Native app integration (Android and iOS)

The CIBA + PAR age-verification integration is a strong, low-friction approach for mobile apps. Your app backend requests an authorization URL via the CIBA + PAR flow, and the app opens that URL in the device’s system browser context (for example, Android Custom Tabs or iOS SFSafariViewController). The user completes the age-verification journey in a standards-compliant browser environment (MitID and other authenticators, URL visibility, EU Wallet / DC-API support, etc.), while the app retrieves the outcome through the CIBA interface.

This pattern avoids common mobile pitfalls around embedded web views and app-switching. It gives you reliable flow control, clear detection of completion, and the ability to close the browser surface when the transaction is finished—even if the user returns to the app without explicitly closing the browser.

### Mobile app CIBA + PAR flow

1. App backend starts the CIBA + PAR age-verification (AV) flow and receives an authorization URL.
2. The mobile app opens the URL in the system browser context (e.g., Custom Tabs / SFSafariViewController).
3. The user completes the AV flow in the system browser with full compatibility and security requirements.
4. (Optional) Status updates are pushed to a registered CIBA callback endpoint.
5. (Optional) The user is automatically returned to the app (app switch).
6. (Optional) The user manually returns to the app.
7. The app (typically via its backend) polls the CIBA endpoint for the final state. Once complete, the app closes the browser surface and proceeds.

### Implementation notes and recommended practices

Use a system browser surface, not an embedded web view.
System browser contexts provide better security properties and compatibility with identity providers and authenticators, and they reduce edge cases related to cookies, session handling, and blocked features.

Design for all return paths.
Users may return via an automatic app switch, by tapping your app icon, or by leaving the browser open in the background. Treat “user returned to app” as a UX signal—not proof of completion. Completion should be driven by CIBA state (push and/or polling).

Prefer push where available; keep polling as a safe fallback.
If you register a callback endpoint, you can reduce perceived latency and polling frequency. Still keep polling to handle network failures, missed callbacks, and devices that suspend background activity.

Handle timeouts, cancellation, and retries cleanly.
Define a clear timeout window for the flow, surface a user-friendly “Try again” option, and distinguish between user cancellation, technical failure, and expired sessions.

Close the browser surface deterministically.
Once the CIBA flow reaches a terminal state (success/failure/cancelled/expired), close the browser UI and continue in-app. This prevents users from accidentally resubmitting or getting stuck on a completed page.

Log correlation identifiers end-to-end.
Persist a stable transaction reference across backend initiation, in-app browser launch, callback handling, and polling. This makes support and debugging dramatically easier—especially for intermittent app-switch issues.
