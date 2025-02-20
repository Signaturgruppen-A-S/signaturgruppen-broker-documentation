---
title: IOS MitID integration
layout: home
parent: App MitID integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 2
---

## IOS MitID integration
On iOS two recommended options are available. 

1. SFSafariViewController - Officially supported by MitID.
2. ASWebAuthenticationSession - Not officially supported by MitID. Recommended by Signaturgruppen.

The SFSafariViewController supports running the Safari browser as part of an iOS app, whereas the ASWebAuthenticationSession is a secure webview browser. 

Signaturgruppen recommend the ASWebAuthenticationSession, as this provides a proper termination hook that allows for a better handling of the UX in the app switch flow. Signaturgruppen consider the risk of a future MitID update to cause errors with the ASWebAuthenticationSession functionality to be small.

> MitID does not test and support anything but the SFSafariViewController, but does allow the usage of ASWebAuthenticationSession, there is no official support from MitID, should ASWebAuthenticationSession stop working with MitID on iOS.

## ASWebAuthenticationSession

This example demonstrates how to use `ASWebAuthenticationSession` in iOS with a custom URL scheme and handle the termination hook when the authentication session ends. This approach allows your app to securely authenticate users via a web interface and then process the callback URL when the authentication flow completes.

---

### Overview

- **ASWebAuthenticationSession**: Provides a secure way to present an authentication web flow to the user.
- **Custom URL Scheme**: Your app must register a URL scheme (e.g., `yourapp://callback`) to receive the authentication response.
- **Termination Hook**: The completion handler of `ASWebAuthenticationSession` acts as a termination hook, called when the session ends (successfully or with an error).

---

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

To enable the MitID app to return to the app, the app needs to implement a Universal link by hosting an apple-app-site-association file in the root of the relavant domain and register a matching associated domain in the app, see <https://developer.apple.com/ios/universal-links/>.
