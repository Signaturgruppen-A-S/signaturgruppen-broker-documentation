---
title: IOS MitID integration
layout: home
parent: App MitID integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 2
---

## IOS MitID integration

On iOS, there are two recommended approaches for integrating MitID:

1. **SFSafariViewController** – Officially supported by MitID.
2. **ASWebAuthenticationSession** – Not officially supported by MitID, but recommended by Signaturgruppen.

While **SFSafariViewController** runs the Safari browser as part of your app, **ASWebAuthenticationSession** provides a secure web view and, importantly, a proper termination hook that improves the UX during the app switch flow. Signaturgruppen considers the risk of future MitID updates affecting ASWebAuthenticationSession functionality to be minimal.

> **Note:** MitID officially tests and supports only SFSafariViewController. Although ASWebAuthenticationSession is permitted, it does not receive official support. If ASWebAuthenticationSession were to stop working with MitID on iOS, MitID would not provide fixes or official guidance.

---

## SFSafariViewController

**SFSafariViewController** is the officially supported browser for MitID flows on iOS. It allows you to embed the Safari browser within your app instance. However, the main challenge with this approach is securely terminating the final step of the MitID app switch flow.

**Recommended Approach:**

- **Final Redirect Handling:**  
  Render a button on the last redirect step (therefore, your OIDC `redirect_uri` must be an `https://` URL). This button should trigger a Universal Links app switch back into your app.

- **App Schemes:**  
  Although using custom URL schemes (e.g., `your-app://`) is possible, it is not recommended because other apps on the device might register the same scheme, posing security concerns.

- **Backend Completion:**  
  An alternative solution is to have your backend complete the OIDC flow at the final browser redirect step, then notify your app via a defined protocol that the flow has completed.

---

## ASWebAuthenticationSession

This section demonstrates how to use `ASWebAuthenticationSession` in iOS with a custom URL scheme and handle the termination hook when the authentication session ends. This approach allows your app to securely authenticate users via a web interface and process the callback URL once the authentication flow completes.

### Overview

- **ASWebAuthenticationSession:**  
  Provides a secure method for presenting an authentication web flow to the user.
  
- **Custom URL Scheme:**  
  Your app must register a URL scheme (e.g., `yourapp://callback`) to receive the authentication response.

- **Termination Hook:**  
  The completion handler in `ASWebAuthenticationSession` acts as a termination hook, which is called when the session ends—whether successfully or with an error.


### Code Example

```swift
import AuthenticationServices
import UIKit

class AuthenticationManager: NSObject {

    private var authSession: ASWebAuthenticationSession?

    func startAuthentication() {
        // The URL for your authentication endpoint.
        guard let authURL = URL(string: "https://your-auth-server.com/auth?client_id=YOUR_CLIENT_ID") else {
            return
        }
        
        // The custom callback URL scheme you registered (e.g., yourapp://callback)
        let callbackScheme = "yourapp"
        
        // Initialize ASWebAuthenticationSession with the authentication URL and callback URL scheme.
        authSession = ASWebAuthenticationSession(url: authURL, callbackURLScheme: callbackScheme) { callbackURL, error in
            // Termination hook: called when the session completes or is cancelled.
            if let error = error {
                print("Authentication error: \(error.localizedDescription)")
                // Handle error if necessary.
                return
            }
            
            guard let callbackURL = callbackURL else {
                print("No callback URL received.")
                return
            }
            
            // Process the callback URL.
            // Example: Extract an authentication token or authorization code from the URL.
            if let queryItems = URLComponents(url: callbackURL, resolvingAgainstBaseURL: false)?.queryItems {
                for item in queryItems {
                    if item.name == "token", let token = item.value {
                        print("Received token: \(token)")
                        // Use the token as needed.
                    }
                }
            }
        }
        
        // Provide the presentation context so that the authentication session is displayed correctly.
        authSession?.presentationContextProvider = self
        
        // Start the authentication session.
        authSession?.start()
    }
}

// Conform to ASWebAuthenticationPresentationContextProviding to specify the presentation anchor.
extension AuthenticationManager: ASWebAuthenticationPresentationContextProviding {
    func presentationAnchor(for session: ASWebAuthenticationSession) -> ASPresentationAnchor {
        // Return the key window of your app.
        return UIApplication.shared.windows.first { $0.isKeyWindow } ?? ASPresentationAnchor()
    }
}
```

## Universal links

To enable the MitID app to return to your app after authentication, implement Universal Links. This requires hosting an apple-app-site-association file at the root of your relevant domain and registering the associated domain in your app.

### Example apple-app-site-association file
```
{
  "applinks": {
    "apps": [
      
    ],
    "details": [
      {
        "appID": "G..X.dk.your.app.production",
        "paths": [
          "/appswitch",
          "/mitidappswitch",
          "/qr"
        ]
      },
      {
        "appID": "G..X.dk.your.app.pp",
        "paths": [
          "/appswitch",
          "/mitidappswitch",
          "/qr"
        ]
      }
    ]
  }
}
```
