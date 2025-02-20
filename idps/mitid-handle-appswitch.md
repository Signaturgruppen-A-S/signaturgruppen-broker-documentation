---
title: App MitID integration
layout: home
parent: MitID
grand_parent: Identity providers
nav_order: 2
---

# Terminology

| **Term** | **Description** |
| --- | --- |
| Android Custom Tabs | Platform specific sandbox of the system default browser, which enables are more seamless and secure browser integration from native apps. <br/><br/> [Android Custom Tabs](https://developer.chrome.com/docs/android/custom-tabs) |
| SFSafariViewController  | Platform specific sandbox of the system default browser, which enables are more seamless and secure browser integration from native apps. <br/><br/> [iOS SFSafariViewController](https://developer.apple.com/documentation/safariservices/sfsafariviewcontroller) |
| ASWebAuthenticationSession  | iOS specific authentication session, the browser is a secure, embedded web view. <br/><br/> [iOS ASWebAuthenticationSesssion](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession) |
| Android App Links | [Android App Links](https://developer.android.com/training/app-links). |
| iOS Universal Links | [iOS Universal Links](https://developer.apple.com/ios/universal-links/). |

# Introduction

This document is a technical guide for Android and iOS app integration to MitID.

It is required that the reader has a basic understanding of the technical details for Android Custom Tabs and App Links and for iOS SFSafariViewController, ASWebAuthenticationSession and Universal Links.

# Supported browsers

MitID only tests and support the browsers that have at least 2% market share on each platform. MitID does not test and support embedded browsers for MitID flows.

On Android it is possible to detect if Chrome or the Samsung browser is available before starting the Custom Tabs intent, it is recommended to target Chrome, as we have seen some issues on some devices with the Samsung browser.

It is a security requirement that the end-user is presented with the address bar from the browser to allow the end-user to verify the **mitid.dk** domain during the MitID flow. It is not allowed to hide the address bar for the Custom Tabs or SFSafariViewController instances. It is required that standard and familiar components are used to achieve this goal with the goal of presenting a familiar and safe MitID experience that allows the end-user to validate that the authentication is taking place on the mitid.dk domain.

If your app is a hybrid web-app serving most content via an inline embedded browser inside the app, then it is still required that the MitID integration is done outside of this component, to ensure proper suppoer and UX for the MitID flow.

## Android browsers
On Android the sole recommendation is to use Custom Tabs targeting the Chrome browser. 

## iOS browser
On iOS two recommended options are available. 

1. SFSafariViewController - Officially supported by MitID.
2. ASWebAuthenticationSession - Not officially supported by MitID. Recommended by Signaturgruppen.

The SFSafariViewController supports running the Safari browser as part of an iOS app, whereas the ASWebAuthenticationSession is a secure webview browser. 

Signaturgruppen recommend the ASWebAuthenticationSession, as this provides a proper termination hook that allows for a better handling of the UX in the app switch flow. Signaturgruppen consider the risk of a future MitID update to cause errors with the ASWebAuthenticationSession functionality to be small.

> MitID does not test and support anything but the SFSafariViewController, but does allow the usage of ASWebAuthenticationSession, there is no official support from MitID, should ASWebAuthenticationSession stop working with MitID on iOS.

# MitID app switch flow overview

In broad terms, the recommended flow can be described in the following way:

- The app backend generates the initial authentication URL
- The app opens the authentication URL in the browser
- The end-user completes the MitID flow by app-switching to the MitID app by clicking the blue button in the MitID flow.
- When the flow is completed in the MitID app, the end-user is redirected back to the specified App Links / Universal Links app-switch url, and is app-switched back to the app.
- **The app then ensures that the Custom Tab / SFSafariViewController / ASWebAuthenticationSession gets into focus again in order to allow the MitID client to finish the MitID flow.**
- The MitID flow will complete in the browser and the end user will be redirected to the "redirect_uri" of the OIDC protocol.
- **The last redirect will not trigger app switch (Universal Link / App Link) as this will not be considered a redirect following user-interaction.**
- The app is notified by the relevant mechanism, that the flow is completed. This step is platform and browser dependent. It is recommended to have a fallback option by rendering a button that the end-user can click which will app switch back into the app, should other automatic mechanisms have failed at this stage. Signaturgruppeen recommends the Custom Tabs postMessage hook on Android, and to use the ASWebAuthenticationSession with a proper termination hook on iOS, in order to be able to handle this last step with success.

### Termination of the browser, returning to the app
The last step in the above app switch protocol overview, is often the most problematic for app switch integrations. It is important to understand that App Links and Universal Links app switch will not trigger at this step, due to the last redirect not being triggered directly from user-interaction such as clicking on a button. It is recommended to render a button at this last step, which then enables proper App Links / Universal Links app switching back into the app, should the user end up here. To avoid this last click on a button for the user, the Custom Tabs postMessage hook and the use of ASWebAuthenticationSession configured with a proper termination hook allows for the control and termination of the last redirect step in the OIDC browser flow. 

If not using ASWebAuthenticationSession, then utilizing app scheme / custom scheme (e.g. your-app://) to terminate the browser flow is not recommended due to security issues with this approach. Further, Chrome on Android has started to require user-interaction in form of an approve dialogue for app scheme app handling, which can prevent the hook from even working properly. 
It is only recommended to utilize app scheme termination of the browser flow when using the ASWebAuthenticationSession iOS component.

## App- / URL- / Custom Schemes vs App Links and Universal Links
The standard way for many app integrations to detect, complete and close the browser-flow in a app switch scenario has been to utilize app schemes, such as app-scheme://oidc-redirect-url, to enable the browser flow to automatically generate an event in the app.
This still works on iOS, but Chrome has begun to require user-interaction for this to work, which for most intends and purposes breaks this on Android. From a security perspective, the app scheme protocol should not be used.

App Links (Android) and Universal Links (iOS) is the recommended mechanism for app switch handling.

# Android 

On Android the Custom Tab can be started as “single-instance” or “single-task”, which affects the behavior. When choosing single-instance, the Custom Tab is started as a separate instance, whereas with single-task, the Custom Tab will be part of the app process. In both scenarios, the Custom Tab will by default stay in the background when returning to the app via App Links app-switching and thus some handling is required here to get the Custom Tab in the foreground again, which is required in order to ensure that the MitID browser flow is able to complete.

In both cases, the browser is still able to complete any flow in the background, if able and when the device is not running in power-saving mode.

The flow can in most cases be completed with the Custom Tab in the background, but there are situations where it is required to have it in the foreground. It is also expected that the MitID client “finish” screen is shown in the browser to the user – so it is a general recommendation to show the Custom Tab until the flows has completed.

## Getting Custom Tabs back to the foreground
Direct app switch back to the app will not automatically get the Android Custom Tab browser to the foreground, but instead puts the app activity that handles the app-switch in the foreground.

If the intention is to have the Custom Tab to the foreground after the app-switch to the app, then it is possible to register an empty activity to handle the app-switch event from the app switch URL given to the MitID app flow and then let this activity close itself upon getting to the foreground based on this event. This will pop this activity and then get the Custom Tab to the foreground. Here it is required to register the Custom Tab as running in “single-task” mode.

## Detection of termination of Custom Tab flow
When the MitID flow completes after the app switch back from the MitID app to the integrating app, there is no inherent way to detect that the flow is completed and to automatically terminate the Custom Tab and return the flow into the app.

App schemes should be avoided and have pitfalls here, App Links will not automatically trigger at this last step, due to the last redirect in the flow not being based on user-interaction. The integrating service should in all scenarios implement the last redirect of the flow to render a button for the end-user which will then trigger an app switch back into the app.
Then, to avoid this last end-user button click, it is possible to setup a postMessage channel between the app and the Custom Tab instance, allowing the rendered page at the last step to notify the app of completion and pass along any required parameters as well, which will enable automatic termination of the flow.

### Custom Tab postMessage channel

When setting up the Custom Tab instance, the app can register a [postMessage channel](https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsSession#requestPostMessageChannel(android.net.Uri)), which allows for JavaScript running at allowed domains to communicate with the app via JS Post Messaging. 

This registration is done in similar ways as for App Link registration, by setting up appropriate settings in the assetlinks.json file in the root of the target domain.

## App links

To enable the MitID app to return to your SP app via App Links, you need to add an intent filter to your main activity in your Manifest.xml, as shown below:

```
<intent-filter android:autoVerify="true">
<action android:name="android.intent.action.VIEW" >
<category android:name="android.intent.category.DEFAULT" >
<category android:name="android.intent.category.BROWSABLE" >

<data
android:scheme="https"
android:host="App Links URL" />
</intent-filter>
```

Furthermore, you need to host an assetlinks.json file on your domain, and the domain should match the host in the manifest file.

See, <https://developer.android.com/training/app-links>.

# iOS

Two options is available for iOS: SFSafariViewController and ASWebAuthenticationSession.

On iOS the SFSafariViewController is part of the app instance and thus when navigating to the MitID app and back to the app using app switch, the SFSafariViewController will be in focus.
The SFSafariViewController do not offer the same control for the app as ASWebAuthenticationSession, but is officially supported by MitID.

ASWebAuthenticationSession is a secure embedded browser, which is setup with a proper termination hook that triggers on the last redirect of the browser flow and returns control to the app.

Signaturgruppen recommends ASWebAuthenticationSession, as this allows for better control and we believe it to be stable across upcoming MitID updates.

## Universal links

To enable the MitID app to return to the app, the app needs to implement a Universal link by hosting an apple-app-site-association file in the root of the relavant domain and register a matching associated domain in the app, see <https://developer.apple.com/ios/universal-links/>.

# App switch for non-app browser flows

MitID has introduced app switching for all mobile browsers and thus enabled that the user is able to app switch to the MitID app directly from a normal browser on a mobile device.

The implementation for this sends the user back to the device browser upon completion of the MitID flow in the MitID app.

This implies that when integrating to MitID from a SP App, it is required to setup the App Links / Universal Links URL as part of the integration to allow the MitID app to redirect to user to the SP App TAB browser instead of sending the user to the system browser.

Note, that if not specifying the return URL, as required, for App Links / Universal Link back to the SP App for the MitID flow, then the user will on some device/browser combinations end up in the device system browser decoupled from the started MitID flow and on some device/browser combinations complete the MitID app flow without being redirected further, but is then able to manually get back to the SP App. Be aware, that this does not work consistently across the different devices and flow combinations.

## Navigation stack issue with MitID app-switch

If app-switch is enabled with an app-switch from the MitID back to the service provider app, then the MitID app will in effect app-switch to, and not back-to, the service provider app and thus create an extra jump in the navigation queue of applications, as opposed to popping one.

This has little effect on the normal user experience, but will affect some app-switch scenarios and is worth considering in conjunction with the decision to enable/disable app-switch back to the service provider app.

If the app-switch chain between apps is longer than the one jump from the service provider app to the MitID app, the flow will work as intended. However, if the user app-switches from app1 to app2 to MitID app, then the app2 is unable to “pop” the navigation stack and return to app1, as in this case the “pop” will result in the user getting back to the MitID app again (the stack is app1 -> app2 -> mitid app -> app2).

As examples of “popping”: on Android some apps close themselves on success, as the NemID app on Android. On iOS you always have a “back arrow” in the upper left corner which gets you back to the caller. In the example above, clicking this back button in app2 after the MitID app-switch will result in getting back to the MitID app again.

# Setting up app switch

To enable app-switch, specify the platform specific return url/bundle ID and the running mobile platform (ios og android).

Example:

```
idp_params=%7B%22mitid%22%3A%7B%22enable_app_switch%22%3A%20true%2C%20%22app_switch_os%22%3A%22ios%22%2C%20%22app_switch_url%22%3A%22https%3A%2F%2Fyour.appswitch.url%2F%22%7D%7D
```

<table><tbody><tr><th><p><strong>Identity Provider parameters (mitid)</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>enable_app_switch</p></td><td><p>Type: bool. Default: false.</p><p>If true, enables MitID app switch for the flow.</p></td></tr><tr><td><p>app_switch_os</p></td><td><p>Type: string</p><p>One of the following values:</p><ul><li>ios</li><li>android</li></ul></td></tr><tr><td><p>app_switch_url</p></td><td><p>Type: String</p><p>Specify the Universal Link / App Links URL that your app can handle.</p><p>For non-signed OIDC requests, the URL must be whitelistet for your OIDC client.</p></td></tr></tbody></table>
