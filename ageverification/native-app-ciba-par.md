---
title: Native Android and iOS integration
layout: home
parent: Age Verification
nav_order: 8
---

# Native app integration from Android and iOS

The CIBA + PAR age-verification integration is a strong, low-friction approach for mobile apps. Your app backend requests an authorization URL via the CIBA + PAR flow, and the app opens that URL in the device’s system browser context (for example, Android Custom Tabs or iOS SFSafariViewController). The user completes the age-verification journey in a standards-compliant browser environment (MitID and other authenticators, URL visibility, EU Wallet / DC-API support, etc.), while the app retrieves the outcome through the CIBA interface.

This pattern avoids common mobile pitfalls around embedded web views and app-switching. It gives you reliable flow control, clear detection of completion, and the ability to close the browser surface when the transaction is finished—even if the user returns to the app without explicitly closing the browser.

## Mobile app CIBA + PAR flow

1. App backend starts the CIBA + PAR age-verification (AV) flow and receives an authorization URL.
2. The mobile app opens the URL in the system browser context (e.g., Custom Tabs / SFSafariViewController).
3. The user completes the AV flow in the system browser with full compatibility and security requirements.
4. (Optional) Status updates are pushed to a registered CIBA callback endpoint
5. (Optional) The user is automatically returned to the app (app switch).
6. (Optional) The user manually returns to the app.
7. The app (typically via its backend) polls the CIBA endpoint for the final state. Once complete, the app closes the browser surface and proceeds.

## Implementation notes and recommended practices

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
