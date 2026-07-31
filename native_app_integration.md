---
title: Native App integration on Android and iOS
layout: home
nav_order: 25
has_children: true
---

# Native App integration on Android and iOS

**Platforms:** Android and iOS/iPadOS  
**Last reviewed:** 31 July 2026

## 1. Purpose

This document describes recommended patterns for browser-mediated authentication and authorization in native Android and iOS applications.

The preferred platform APIs are:

- **Android:** AndroidX **Auth Tab** where supported, with **Custom Tabs** and verified **Android App Links** as fallback.
- **iOS/iPadOS:** **`ASWebAuthenticationSession`**, preferably with an HTTPS callback matched by host and path where the deployment target supports it, with **Universal Links** as the general recovery mechanism.

The design goal is:

1. Keep credentials and identity-provider pages inside a browser-controlled context.
2. Return the result to the initiating application session without depending on a generic browser-to-app launch where possible.
3. Preserve a robust, user-activated fallback when automatic app switching is suppressed.
4. Maintain OAuth/OIDC security independently of platform handoff behavior.

> Terminology note: `ASWebAuthenticationSession` is an Apple API. The closest Android equivalent is Android Auth Tab, not an Android implementation of `ASWebAuthenticationSession`.

---

## 2. Recommended defaults

| Concern | Android default | iOS default |
|---|---|---|
| Browser surface | Auth Tab | `ASWebAuthenticationSession` |
| Legacy fallback | Custom Tab | SFSafariViewController / System browser authentication session |
| Preferred redirect | Claimed HTTPS URI | Claimed HTTPS URI |
| App/domain association | Digital Asset Links | Associated Domains and AASA |
| Primary result delivery | Auth Tab Activity Result | Session completion handler |
| Generic link fallback | Android App Link | Universal Link |
| OAuth flow | Authorization Code + PKCE | Authorization Code + PKCE |
| Browser data mode | Shared browser session | Shared browser session |
| Ephemeral/private mode | Only for an explicit product/privacy requirement | Only for an explicit product/privacy requirement |
| Last-resort recovery | User-activated **Open app** link/button | User-activated **Open app** link/button |

Use a custom URL scheme only when a claimed HTTPS redirect is unavailable or incompatible with a required provider. A reverse-domain custom scheme is preferable to a short generic scheme, but it is still not an authenticated global namespace.

---

## 3. Common architecture

A normal native authorization transaction should look like this:

```text
Native app
  -> creates local transaction
  -> launches platform browser-authentication API
  -> authorization server / broker
  -> optional upstream IdP or second application
  -> authorization callback URI
  -> platform authentication session returns callback to app
  -> app validates transaction
  -> app redeems authorization code with PKCE
```

The local transaction should contain at least:

- A high-entropy, single-use `state`.
- A PKCE `code_verifier` and corresponding S256 `code_challenge`.
- An OIDC `nonce`, when OpenID Connect is used.
- The expected issuer or authorization server.
- The exact expected redirect URI.
- Creation and expiration times.
- A stable local transaction identifier.
- A status that can be changed atomically from pending to processing to completed.

Do not depend on a single in-memory Activity, View Controller, or callback closure. The operating system may suspend or terminate the application while the user is authenticating.

---

## 4. Android

### 4.1 Preferred API: Android Auth Tab

Android Auth Tab is a specialized Custom Tab for authentication. The application tells the browser which redirect is expected. When that redirect is reached in the same Auth Tab session, the browser returns the callback URI through Android's Activity Result API.

This changes the final transition from:

```text
Web navigation
  -> generic Android intent resolution
  -> application Activity
```

to:

```text
Web navigation inside Auth Tab
  -> Auth Tab recognizes configured callback
  -> Activity Result callback
```

This is particularly valuable after multi-domain redirect chains because the final handoff is completion of a known authentication session rather than an unsolicited request by a web page to open an application.

Use AndroidX Browser 1.9.0 or later:

```kotlin
implementation("androidx.browser:browser:1.9.0")
```

Register the launcher during Activity or Fragment initialization:

```kotlin
private val authTabLauncher =
    AuthTabIntent.registerActivityResultLauncher(this) { result ->
        when (result.resultCode) {
            AuthTabIntent.RESULT_OK ->
                processAuthorizationCallback(result.resultUri)

            AuthTabIntent.RESULT_CANCELED ->
                handleCancellation()

            AuthTabIntent.RESULT_VERIFICATION_FAILED ->
                handleRedirectAssociationFailure()

            AuthTabIntent.RESULT_VERIFICATION_TIMED_OUT ->
                handleRedirectAssociationTimeout()
        }
    }
```

Launch with an HTTPS callback host and path:

```kotlin
val authTabIntent = AuthTabIntent.Builder().build()

authTabIntent.launch(
    authTabLauncher,
    authorizationUri,
    "auth.example.com",
    "/mobile/oauth/callback"
)
```

The application should still validate the returned URI exactly. Auth Tab callback matching does not replace protocol validation.

### 4.2 Detect capability; do not infer it from Custom Tab support

Auth Tab is a browser capability. A browser can support Custom Tabs without supporting Auth Tab.

Use the AndroidX capability API, such as `CustomTabsClient.isAuthTabSupported(...)`, instead of checking a browser brand or hard-coding a Chrome version.

The integration must support two result paths:

1. **Auth Tab path:** callback through the Activity Result API.
2. **Fallback path:** callback through an App Link/custom-scheme intent after an ordinary Custom Tab fallback.

Both paths should call one idempotent callback processor:

```kotlin
fun processAuthorizationCallback(uri: Uri?) {
    if (uri == null) return

    authorizationCoordinator.process(uri)
}
```

Invoke it from the Auth Tab result and from relevant Activity intent handlers:

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    processAuthorizationCallback(intent?.data)
}

override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    setIntent(intent)
    processAuthorizationCallback(intent.data)
}
```

The coordinator must reject duplicate or already-consumed transactions.

### 4.3 Android App Links

Use a verified HTTPS callback such as:

```text
https://auth.example.com/mobile/oauth/callback
```

Configure an App Link intent filter:

```xml
<activity
    android:name=".OAuthCallbackActivity"
    android:exported="true">

    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />

        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <data
            android:scheme="https"
            android:host="auth.example.com"
            android:pathPrefix="/mobile/oauth/callback" />
    </intent-filter>
</activity>
```

Publish Digital Asset Links at:

```text
https://auth.example.com/.well-known/assetlinks.json
```

Example:

```json
[
  {
    "relation": [
      "delegate_permission/common.handle_all_urls"
    ],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.mobile",
      "sha256_cert_fingerprints": [
        "AA:BB:CC:..."
      ]
    }
  }
]
```

Operational requirements:

- Include the certificate fingerprint of the application actually installed on the device.
- Account for Google Play App Signing; the Play signing certificate can differ from the local upload certificate.
- Separate production, preproduction, internal, and debug associations deliberately.
- Serve the file over valid HTTPS without authentication.
- Avoid redirects for `/.well-known/assetlinks.json`.
- Keep manifest host/path constraints and server association data aligned.
- Verify behavior on supported Android API levels and browsers.
- Do not define overlapping matching filters in multiple activities.

### 4.4 Preserve the original Auth Tab session

For flows that temporarily invoke another application, the best sequence is:

```text
App A
  -> Auth Tab A
  -> web broker
  -> App B
  -> returns to Auth Tab A
  -> browser reaches App A's configured callback
  -> Auth Tab A completes through Activity Result
```

Avoid:

```text
App A
  -> Auth Tab A
  -> App B
  -> App B opens a new Custom Tab B
  -> callback occurs in Custom Tab B
```

Auth Tab A cannot generally intercept navigation inside an unrelated browser surface. The final return then depends on generic App Link behavior.

Document the return contract for App B explicitly. It should resume or return to the browser/authentication session that initiated the app switch, rather than starting a new browser task unless that behavior is unavoidable.

### 4.5 Android custom-scheme fallback

When HTTPS callbacks cannot be used:

```text
com.example.mobile:/oauth/callback
```

Use a reverse-domain scheme. Do not assume uniqueness merely because the scheme resembles a domain; another application can register the same scheme.

Auth Tab's session-bound return improves callback delivery, but the application must still validate `state`, PKCE, issuer, redirect URI, and all returned tokens.

---

## 5. iOS and iPadOS

### 5.1 Preferred API: `ASWebAuthenticationSession`

`ASWebAuthenticationSession` provides a browser-controlled authentication surface and returns a matching callback URL to the initiating session's completion handler.

Retain the session strongly until it completes:

```swift
import AuthenticationServices

final class AuthenticationCoordinator: NSObject,
    ASWebAuthenticationPresentationContextProviding {

    private var session: ASWebAuthenticationSession?

    func start(
        authorizationURL: URL,
        completion: @escaping (Result<URL, Error>) -> Void
    ) {
        let callback = ASWebAuthenticationSession.Callback.https(
            host: "auth.example.com",
            path: "/mobile/oauth/callback"
        )

        let session = ASWebAuthenticationSession(
            url: authorizationURL,
            callback: callback
        ) { callbackURL, error in
            defer { self.session = nil }

            if let callbackURL {
                completion(.success(callbackURL))
                return
            }

            completion(.failure(
                error ?? AuthenticationError.missingCallback
            ))
        }

        session.presentationContextProvider = self

        // Shared browser state is the best default for SSO and passkeys.
        session.prefersEphemeralWebBrowserSession = false

        self.session = session

        guard session.start() else {
            self.session = nil
            completion(.failure(AuthenticationError.failedToStart))
            return
        }
    }

    func presentationAnchor(
        for session: ASWebAuthenticationSession
    ) -> ASPresentationAnchor {
        // Return the active foreground window for the current scene.
        UIApplication.shared.connectedScenes
            .compactMap { $0 as? UIWindowScene }
            .filter { $0.activationState == .foregroundActive }
            .flatMap(\.windows)
            .first { $0.isKeyWindow }
            ?? ASPresentationAnchor()
    }
}

enum AuthenticationError: Error {
    case missingCallback
    case failedToStart
}
```

For deployments using an older callback API, initialize the session with a custom callback scheme:

```swift
let session = ASWebAuthenticationSession(
    url: authorizationURL,
    callbackURLScheme: "com.example.mobile"
) { callbackURL, error in
    // Process result.
}
```

Prefer the HTTPS host/path callback API where it is available and compatible with the deployment target and identity provider.

### 5.2 Presentation context

Always set `presentationContextProvider`.

For multi-scene applications, do not blindly return the first window in the process. Return the active window belonging to the scene that initiated authentication. A production coordinator should receive or retain that scene/window explicitly.

Only one interactive authentication operation should normally be active per user-facing scene. Prevent double taps and concurrent sessions unless transaction routing has been deliberately designed for it.

### 5.3 Browser session data

The best default is:

```swift
session.prefersEphemeralWebBrowserSession = false
```

This preserves browser SSO, remembered account choices, passkeys, and other shared browser state.

Set it to `true` only for an explicit requirement such as:

- The user selected a private or “use another account without remembering it” mode.
- The authorization server requires isolation for a well-defined reason.
- A product/privacy policy intentionally rejects shared SSO state.

Do not use ephemeral mode as a general-purpose privacy improvement without considering the loss of SSO and credential continuity. It is a request to the browser environment, not a replacement for server-side logout or account selection.

### 5.4 Universal Links

Configure the Associated Domains entitlement:

```text
applinks:auth.example.com
```

Publish the Apple App Site Association file at an accepted location, commonly:

```text
https://auth.example.com/.well-known/apple-app-site-association
```

A simplified example:

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": [
          "TEAMID.com.example.mobile"
        ],
        "components": [
          {
            "/": "/mobile/oauth/callback*"
          },
          {
            "/": "/mobile/open-app*"
          }
        ]
      }
    ]
  }
}
```

Operational requirements:

- Serve the file over valid HTTPS.
- Do not require cookies, authentication, or client certificates.
- Use the exact Apple Team ID and bundle identifier.
- Keep callback and recovery paths narrowly scoped.
- Test installed production-signed builds, not only simulator or debug builds.
- Treat AASA caching and deployment propagation as operational concerns.
- Retain the normal Universal Link handler as a fallback even when `ASWebAuthenticationSession` receives the callback directly.

A general Universal Link may arrive through scene/application continuation APIs. Route it into the same idempotent authorization coordinator used by the session completion handler.

### 5.5 Custom URL schemes on iOS

Apple recommends Universal Links over custom URL schemes for links that should be uniquely associated with an application. Multiple applications can register the same custom scheme.

When a custom scheme is required:

```text
com.example.mobile:/oauth/callback
```

Use a reverse-domain identifier and keep the path exact. `ASWebAuthenticationSession` binds callback delivery to the initiating session, but the scheme itself is not globally owned or verified.

---

## 6. Why automatic app switching sometimes fails

A browser may decline to open an external application when the final link is reached through a chain of automatic redirects, especially when the navigation:

- Crosses several domains.
- Crosses between browser and native applications.
- Occurs without a recent user gesture.
- Is initiated by script or a timed redirect.
- Appears to be an unsolicited attempt to leave the browser.
- Happens in a browser surface other than the original authentication session.
- Targets an unverified or incorrectly associated link.
- Is affected by user link-opening preferences or platform state.

Example:

```text
App
  -> browser/auth session
  -> broker.example
  -> idp.example
  -> broker.example/callback
  -> auth.example.com/mobile/oauth/callback
```

Auth Tab and `ASWebAuthenticationSession` are designed to catch the configured callback inside their own session. That is the preferred path.

However, if the flow escapes the original session, falls back to a normal browser, or reaches a general completion page, the browser may render the HTTPS target instead of switching to the app.

This must be treated as an expected recovery condition, not as an impossible edge case.

---

## 7. The user-activated “Open app” recovery pattern

### 7.1 Purpose

The callback or completion endpoint should be able to render a safe recovery page containing a real link/button:

```text
Authentication completed

[Open the app]
```

The button points to a verified Universal Link or Android App Link. The user's tap creates a direct user activation and is generally more reliable than another automatic redirect.

This pattern is a **fallback**, not the primary callback design.

### 7.2 Recommended behavior

1. The authorization server or broker completes its server-side processing.
2. It attempts the normal platform/session callback.
3. When the callback is handled as ordinary web navigation, the server renders a completion page.
4. The page exposes a normal `<a href="...">` link styled as a button.
5. The target is a claimed HTTPS Universal Link/App Link.
6. The page also explains what to do when the application is not installed or the link remains in the browser.
7. The transaction cannot be replayed by repeatedly pressing the button.

Use an actual anchor as the primary control:

```html
<a
  class="open-app-button"
  href="https://auth.example.com/mobile/open-app?t=ONE_TIME_HANDLE">
  Open the app
</a>
```

A normal anchor preserves platform link semantics better than a JavaScript-only click handler.

### 7.3 Do not put sensitive protocol data in the recovery URL

Avoid placing authorization codes, tokens, PII, or reusable secrets directly in a link that can be copied, logged, synced, or leaked through browser history.

Prefer a short-lived, one-time opaque handle:

```text
https://auth.example.com/mobile/open-app?t=ONE_TIME_HANDLE
```

The native app redeems the handle through a protected backchannel or uses it to locate a pending transaction. The server should bind it to:

- The original authorization transaction.
- The intended client/application.
- A short expiration time.
- Single use.
- The expected redirect or return context.

Whether this handle replaces or merely resumes the normal OAuth callback depends on the architecture. Do not invent a second, weaker authorization protocol solely for the fallback page.

### 7.4 Avoid redirect loops

The same HTTPS URL can be both a website URL and an app-associated URL. Design separate paths when practical:

```text
/mobile/oauth/callback   Session callback
/mobile/open-app         User-activated recovery link
/mobile/complete         Human-readable web page
```

The web behavior should terminate predictably when app switching does not happen. Do not repeatedly redirect between `/complete` and `/open-app`.

A safe page may perform one best-effort automatic attempt only when the flow has a clear reason to do so, but it must quickly settle on a stable page with an explicit button. Never use an endless timer loop.

### 7.5 Suggested recovery page

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Return to the app</title>
</head>
<body>
  <main>
    <h1>Authentication completed</h1>
    <p>Return to the app to continue.</p>

    <p>
      <a
        href="https://auth.example.com/mobile/open-app?t=ONE_TIME_HANDLE"
        rel="external">
        Open the app
      </a>
    </p>

    <p>
      If the app does not open, switch to it manually. You can safely close
      this page after the app confirms completion.
    </p>
  </main>
</body>
</html>
```

Keep the page usable without JavaScript.

### 7.6 Optional custom-scheme fallback

A separate custom-scheme button can be offered only when the threat model and installation model permit it:

```html
<a href="com.example.mobile:/resume?t=ONE_TIME_HANDLE">
  Try opening the app
</a>
```

This has weaker ownership guarantees than a verified HTTPS link. It should not silently receive the same level of trust as a Universal Link/App Link.

Do not automatically cascade from HTTPS to a custom scheme using hidden iframes or aggressive timers. Such techniques are fragile, can trigger browser protections, and complicate accessibility and security analysis.

### 7.7 Button text

Use action-oriented text:

- **Open the app**
- **Return to Example App**
- **Continue in Example App**

Avoid misleading text such as “Authentication successful” when the app has not yet validated the response or redeemed the authorization code.

A more precise state model is:

```text
Browser/server step completed
  !=
Native application has validated and completed sign-in
```

---

## 8. Callback validation

Every callback path must converge on the same validation logic.

Before redeeming a code:

1. Parse the URI with a strict URI parser.
2. Require the expected scheme.
3. Require the exact expected host.
4. Require the expected path.
5. Reject unexpected user-info, fragments, ports, or duplicate security-sensitive parameters.
6. Require a pending, unexpired local transaction.
7. Compare `state` using an exact comparison.
8. Atomically mark the transaction as processing.
9. Process OAuth error responses intentionally.
10. Redeem the code once, using the original PKCE verifier.
11. Validate the authorization server/issuer relationship.
12. Validate OIDC ID-token signature, issuer, audience, nonce, and time claims.
13. Mark the transaction completed.
14. Reject later duplicate deliveries.

Do not accept a callback merely because it arrived through:

- Auth Tab.
- `ASWebAuthenticationSession`.
- A verified App Link.
- A Universal Link.
- A known custom scheme.

Platform routing and OAuth/OIDC validation solve different problems.

---

## 9. App lifecycle and idempotency

The same logical callback can be observed through more than one platform route during fallback or lifecycle transitions.

Examples:

- Android Auth Tab result and an Activity intent.
- iOS session completion and a Universal Link continuation.
- Application resumed after process recreation.
- User presses the recovery button more than once.
- Browser retries navigation.

The callback processor should use an atomic transaction state:

```text
Pending -> Processing -> Completed
          \-> Failed
```

A callback for a completed transaction should return the already-known result or be ignored safely. It must not redeem the same authorization code again.

Persist only what is necessary and protect sensitive material. PKCE verifiers and similar transaction secrets should not be written to ordinary logs, analytics events, crash metadata, URLs, or browser storage.

---

## 10. Error handling

Distinguish at least:

- User cancellation.
- Browser/authentication session failed to start.
- Redirect association verification failed.
- Redirect association verification timed out.
- Invalid or missing callback URI.
- State mismatch.
- Expired or unknown transaction.
- OAuth authorization error.
- Authorization-code redemption failure.
- OIDC validation failure.
- Network loss after browser completion.
- App installed but associated link did not open it.
- App not installed.
- Duplicate callback.

Do not convert all failures into “login cancelled.” That hides security and deployment defects.

For association failures, telemetry should record non-sensitive diagnostics such as:

- Platform and OS version.
- Application version.
- Browser package/version on Android.
- Callback host and normalized path.
- Auth Tab capability result.
- Association verification outcome.
- Whether recovery-button navigation was used.
- Transaction phase, without `code`, `state`, tokens, or full query strings.

---

## 11. Testing matrix

Test real devices and production-like signed builds.

### Android

- Supported Chrome with Auth Tab.
- A browser with Custom Tabs but no Auth Tab.
- App Link verified.
- App Link unverified.
- Google Play-signed production build.
- Internal/debug signing key.
- Application process killed during authentication.
- Multiple redirects across domains.
- Temporary switch to another native application.
- Other application returns to original Auth Tab.
- Other application opens a new Custom Tab.
- Recovery button tapped.
- Recovery button tapped repeatedly.
- App absent.
- User disables supported-link opening for the app.
- Offline state after authorization but before code redemption.

Useful verification commands include Android's App Link verification tooling, for example:

```bash
adb shell pm verify-app-links --re-verify com.example.mobile
adb shell pm get-app-links com.example.mobile
```

### iOS/iPadOS

- Current supported iOS versions.
- Shared browser session.
- Requested ephemeral session.
- Universal Link opens installed app.
- Universal Link remains in browser.
- AASA changed or unavailable.
- Multiple scenes/windows.
- Application terminated during authentication.
- Recovery button tapped.
- App absent.
- Custom-scheme fallback, when supported.
- Passkey/password-manager and existing SSO behavior.
- Cancellation from the system authentication UI.
- Network loss before and after callback.

Do not treat “opening the URL directly from a developer tool works” as proof that the full redirected authentication chain will behave identically.

---

## 12. Recommended server endpoint structure

A clean structure separates protocol, recovery, and informational behavior:

```text
/oauth/authorize
/mobile/oauth/callback
/mobile/complete
/mobile/open-app
/mobile/resume
```

Example responsibilities:

### `/mobile/oauth/callback`

- Exact registered OAuth redirect URI.
- Completes or records the browser-side transaction.
- Is matched by Auth Tab or `ASWebAuthenticationSession`.
- Can render/redirect to a stable completion page if treated as web content.

### `/mobile/complete`

- Human-readable status.
- No reusable secrets.
- Contains the user-activated **Open app** button.
- Does not claim native completion before the app confirms it.

### `/mobile/open-app`

- Verified Universal Link/App Link path.
- Carries only a short-lived opaque handle where needed.
- Opens the app when link association and platform policy allow it.
- Has a safe website fallback when the app is absent or switching is suppressed.

### `/mobile/resume`

- Optional application-side semantic route.
- Resolves the one-time handle to the pending transaction.
- Does not weaken the normal OAuth/OIDC validation requirements.

---

## 13. Anti-patterns

Avoid:

- Embedded WebViews for third-party identity-provider authentication.
- Implicit Flow or tokens returned in URL fragments.
- Authorization Code Flow without PKCE.
- Wildcard redirect URIs.
- Generic callback schemes such as `login://`.
- Assuming a redirect chain always has permission to open an app.
- Assuming Custom Tab support implies Auth Tab support.
- Opening a new browser surface after returning from a second app when the original authentication session can be resumed.
- Keeping transaction state only in memory.
- Treating `state` as optional because the platform session is bound.
- Putting access tokens, ID tokens, authorization codes, or PII into recovery links.
- JavaScript-only recovery controls.
- Endless automatic deep-link retry loops.
- Hidden iframe custom-scheme launch tricks.
- Marking authentication complete before the native app validates the callback.
- Logging complete callback URLs.
- Using ephemeral sessions by default without a product requirement.
- Supporting only the session callback and forgetting App Link/Universal Link fallback routes.
- Supporting only generic deep links and not using session-bound callbacks.

---

## 14. Reference flow

```text
1. App creates transaction:
   state + nonce + PKCE + expected redirect + expiry

2. App launches:
   Android: Auth Tab
   iOS: ASWebAuthenticationSession

3. User completes browser or cross-app interaction.

4. Preferred completion:
   Original platform auth session recognizes callback and returns URI directly.

5. Fallback completion:
   Callback is shown as a web page.

6. Page displays:
   [Open the app]

7. User taps verified HTTPS Universal Link/App Link.

8. App receives link through its generic link handler.

9. Both direct and fallback paths call the same idempotent coordinator.

10. App validates state and callback, redeems code with PKCE,
    validates OIDC response, and completes the local transaction.
```

---

## 15. Final implementation checklist

### Shared

- [ ] External browser-controlled authentication surface.
- [ ] Authorization Code Flow with PKCE S256.
- [ ] High-entropy, single-use `state`.
- [ ] OIDC `nonce`.
- [ ] Exact registered redirect URI.
- [ ] Claimed HTTPS callback where possible.
- [ ] Persisted, expiring transaction state.
- [ ] Idempotent callback processor.
- [ ] Strict URI and token validation.
- [ ] No sensitive values in logs or recovery links.
- [ ] Stable web completion page.
- [ ] Real user-activated **Open app** anchor.
- [ ] No redirect loops.
- [ ] Real-device and production-signing tests.

### Android

- [ ] AndroidX Auth Tab as primary.
- [ ] Runtime Auth Tab capability detection.
- [ ] Custom Tab fallback supported.
- [ ] Android App Link configured.
- [ ] Correct Digital Asset Links signing fingerprints.
- [ ] Activity Result and Activity intent routed to one coordinator.
- [ ] Cross-app flow returns to original Auth Tab where possible.

### iOS/iPadOS

- [ ] `ASWebAuthenticationSession` retained strongly.
- [ ] Correct `presentationContextProvider`.
- [ ] HTTPS host/path callback API where supported.
- [ ] Universal Link and Associated Domains configured.
- [ ] Correct AASA Team ID, bundle ID, and paths.
- [ ] Session completion and Universal Link continuation routed to one coordinator.
- [ ] Shared browser session used by default.
- [ ] Multi-scene presentation and callback behavior tested.

---

## 16. Sources

1. Chrome for Developers, **Simplify authentication using Auth Tab**:  
   https://developer.chrome.com/docs/android/custom-tabs/guide-auth-tab

2. Android Developers, **`AuthTabIntent` API reference**:  
   https://developer.android.com/reference/androidx/browser/auth/AuthTabIntent

3. Android Developers, **`CustomTabsClient.isAuthTabSupported`**:  
   https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsClient

4. Android Developers, **About Android App Links**:  
   https://developer.android.com/training/app-links/about

5. Android Developers, **Verify Android App Links**:  
   https://developer.android.com/training/app-links/verify-applinks

6. Apple Developer Documentation, **`ASWebAuthenticationSession`**:  
   https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession

7. Apple Developer Documentation, **`ASWebAuthenticationSession.Callback`**:  
   https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession/callback

8. Apple Developer Documentation, **Supporting Universal Links in your app**:  
   https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app

9. Apple Developer Documentation, **Defining a custom URL scheme for your app**:  
   https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app

10. IETF RFC 8252, **OAuth 2.0 for Native Apps**:  
    https://www.rfc-editor.org/rfc/rfc8252

11. IETF RFC 9700, **Best Current Practice for OAuth 2.0 Security**:  
    https://www.rfc-editor.org/rfc/rfc9700
