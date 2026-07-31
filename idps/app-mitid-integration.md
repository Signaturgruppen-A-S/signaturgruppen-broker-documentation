---
title: App MitID Integration
layout: home
parent: MitID
grand_parent: Identity providers
has_children: true
nav_order: 2
---

# Android and iOS native app guidelines

* Updated 31-07-2026
See the [updated guide for app integrations for Android and iOS](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/native_app_integration.html)

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
  It is essential that users see the browser’s address bar during the MitID flow. This visibility confirms the **mitid.dk** domain and ensures a secure and familiar experience. The address bar must remain visible in both Custom Tabs and ASWebAuthenticationSession instances.

- **Hybrid Web-Apps:**  
  Even if your app predominantly uses an embedded web view, the MitID integration must occur outside this component to guarantee proper user experience and security.

# Configuring App Switch for OIDC with Signaturgruppen Broker

Signaturgruppen Broker offers two methods for enabling app switch during MitID flows:

1. **Flow-Specific Parameters:**  
   Define parameters for each OIDC flow, including the operating system and a whitelisted app switch URL.

2. **Client Default URL:**  
   Set a default app switch URL in the Broker Admin UI for all flows associated with the OIDC client.  
   *Note: Providing the `idp_params` parameter as specified below will override this default setting.*

## Client Default App Switch URL

Configure a single default app switch URL in the Signaturgruppen Broker Admin UI. This URL will be used automatically for every flow for that client unless the OIDC parameter `idp_params` is provided.

## OIDC Parameter

An OIDC parameter named `idp_params` can be provided which consists of a url encoded JSON object, providing this object prevents the usage of a Default App Switch URL provided in the portal.

For example:

```
idp_params=%7B%22mitid%22%3A%7B%22enable_app_switch%22%3A%20true%2C%20%22app_switch_os%22%3A%22ios%22%2C%20%22app_switch_url%22%3A%22https%3A%2F%2Fyour.appswitch.url%2F%22%7D%7D
```

| **Parameter**          | **Description**                                                                                                                                                            |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **enable_app_switch**  | *Type:* Boolean. Whether app switching is activated for the MitID flow. If set to true, the other two fields are required.                                                                           |
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
