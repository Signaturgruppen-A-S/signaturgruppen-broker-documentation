---
title: App MitID Integration
layout: home
parent: MitID
grand_parent: Identity Providers
has_children: true
nav_order: 2
---

# Key Terminology

| **Term**                        | **Description**                                                                                                                                                                                                                                                                                                                       |
|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Android Custom Tabs**         | A secure, sandboxed instance of the system browser on Android that offers a seamless integration with native apps. [Learn more](https://developer.chrome.com/docs/android/custom-tabs)                                                                                                                                      |
| **SFSafariViewController**      | A dedicated Safari browser view on iOS that provides secure and integrated browsing experiences for native applications. [Learn more](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller)                                                                                                           |
| **ASWebAuthenticationSession**  | An iOS-specific authentication session that uses an embedded web view for secure authentication processes. [Learn more](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession)                                                                                                              |
| **Android App Links**           | Mechanism for handling links in Android that allows an app to verify and handle URLs specific to its domain. [Learn more](https://developer.android.com/training/app-links)                                                                                                                                                      |
| **iOS Universal Links**         | A feature on iOS that enables a seamless handover from a web link to an installed app, enhancing user experience and security. [Learn more](https://developer.apple.com/ios/universal-links/)                                                                                                                                   |

# Overview

This guide provides a technical reference for integrating MitID with both Android and iOS applications. It assumes that readers are familiar with Android Custom Tabs and App Links as well as iOS’s SFSafariViewController, ASWebAuthenticationSession, and Universal Links.

# Supported Browsers

MitID only supports browsers that hold at least a 2% market share on each platform. Embedded browsers are not supported for MitID flows.

- **Android:**  
  You can detect the presence of Chrome or Samsung Browser before launching the Custom Tabs intent. Chrome is the preferred target because issues have been noted with Samsung Browser on certain devices.

- **Security Requirement:**  
  It is essential that users see the browser’s address bar during the MitID flow. This visibility confirms the **mitid.dk** domain and ensures a secure and familiar experience. The address bar must remain visible in both Custom Tabs and SFSafariViewController instances.

- **Hybrid Web-Apps:**  
  Even if your app predominantly uses an embedded web view, the MitID integration must occur outside this component to guarantee proper user experience and security.

# MitID App Switch Flow Overview

The recommended MitID integration follows these steps:

1. **Initiate the Flow:**  
   The app’s backend creates the initial authentication URL.

2. **Open in Browser:**  
   The URL is then opened in the system browser (via Android Custom Tabs, SFSafariViewController, or ASWebAuthenticationSession).

3. **User Interaction:**  
   The user completes the MitID flow and taps the blue button to switch from the browser to the MitID app.

4. **App Switch Back:**  
   After finishing in the MitID app, the user is redirected to the preconfigured App Links or Universal Links URL, which triggers a switch back to the original app.

5. **Completion in Browser:**  
   The browser regains focus so that the MitID client can finalize the flow. Ultimately, the process ends at the "redirect_uri" as defined by the OIDC protocol:
   - **Android:**  
     The redirect URI should be an HTTPS page under your domain, where JavaScript’s postMessage API communicates back to the app or backend.
   - **iOS with SFSafariViewController:**  
     Termination is challenging because using a custom scheme (e.g., your-app://) can expose security vulnerabilities. Instead, a button should be provided to trigger the app switch.
   - **iOS with ASWebAuthenticationSession:**  
     The session automatically ends when it detects a redirect using a secure, app-bound scheme (e.g., your-app://), ensuring a secure and smooth transition without extra clicks.

### Handling the Final Redirect

The last step—returning from the browser to the app—is often problematic since App Links and Universal Links do not trigger automatically without user interaction. To address this:
- **Recommended Approach:**  
  Include a button on the final redirect page to allow the app to regain focus.
- **Alternatives:**  
  Using Custom Tabs’ postMessage hook or ASWebAuthenticationSession’s termination hook can eliminate the need for an extra click.
- **Note on Custom Schemes:**  
  Relying on custom scheme redirection without ASWebAuthenticationSession is discouraged due to security risks and potential issues with Chrome on Android, which may prompt a user-approval dialog.

# Configuring App Switch for OIDC with Signaturgruppen Broker

Signaturgruppen Broker offers two methods for enabling app switch during MitID flows:

1. **Flow-Specific Parameters:**  
   Define parameters for each OIDC flow, including the operating system and a whitelisted app switch URL.

2. **Client Default URL:**  
   Set a default app switch URL in the Broker Admin UI for all flows associated with the OIDC client.  
   *Note: Specific OIDC parameters provided later will override this default setting.*

## Client Default App Switch URL

Configure a single default app switch URL in the Signaturgruppen Broker Admin UI. This URL will be used automatically for every app switch flow unless overridden by specific OIDC parameters.

## OIDC Parameters

For example:

```
idp_params=%7B%22mitid%22%3A%7B%22enable_app_switch%22%3A%20true%2C%20%22app_switch_os%22%3A%22ios%22%2C%20%22app_switch_url%22%3A%22https%3A%2F%2Fyour.appswitch.url%2F%22%7D%7D
```

| **Parameter**          | **Description**                                                                                                                                                            |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **enable_app_switch**  | *Type:* Boolean (Default: false). When set to true, app switching is activated for the MitID flow.                                                                          |
| **app_switch_os**      | *Type:* String. Acceptable values: `ios` or `android`.                                                                                                                    |
| **app_switch_url**     | *Type:* String. The Universal Link or App Links URL that the app is set to handle. For non-signed OIDC requests, this URL must be whitelisted for your OIDC client.  |

# App Switch in Non-App Browser Flows

MitID now supports app switching directly from any mobile browser. With this implementation, after completing the MitID flow in the MitID app, the user is sent back to the originating device browser rather than the system browser.

- **Integration Requirement:**  
  When integrating MitID into a Service Provider (SP) app, you must configure the App Links or Universal Links URL so that the MitID app can redirect the user back to the correct in-app browser tab.

- **Without Proper Configuration:**  
  Failing to specify the return URL may result in inconsistent behavior. Some devices might launch the system browser or leave the user stranded within the MitID app flow, requiring manual navigation back to the SP app.

# Navigation Stack Considerations

When using app switching to return from the MitID app to the service provider app, an extra step is introduced into the navigation stack. For example:
- **Scenario:**  
  If a user switches from app1 → app2 → MitID app, then app2 might be unable to properly “pop” the stack to return to app1, instead cycling back to the MitID app.
- **Platform Differences:**  
  - On **Android**, some apps may close themselves upon successful authentication (similar to the NemID app).
  - On **iOS**, the back arrow is used for navigation; however, it might return the user to the MitID app rather than the original calling app if the navigation stack includes multiple jumps.

This behavior should be considered when deciding whether to enable app switching back to the service provider app.
