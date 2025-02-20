---
title: Android MitID integration
layout: home
parent: App MitID integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 1
---

# Android MitID integration
On Android the sole recommendation is to use Custom Tabs targeting the Chrome browser. 

On Android the Custom Tab can be started as “single-instance” or “single-task”, which affects the behavior. When choosing single-instance, the Custom Tab is started as a separate instance, whereas with single-task, the Custom Tab will be part of the app process. In both scenarios, the Custom Tab will by default stay in the background when returning to the app via App Links app-switching and thus some handling is required here to get the Custom Tab in the foreground again, which is required in order to ensure that the MitID browser flow is able to complete.

In both cases, the browser is still able to complete any flow in the background, if able and when the device is not running in power-saving mode.

The flow can in most cases be completed with the Custom Tab in the background, but there are situations where it is required to have it in the foreground. It is also expected that the MitID client “finish” screen is shown in the browser to the user – so it is a general recommendation to show the Custom Tab until the flows has completed.

## Getting Custom Tab back to the foreground
Direct app switch back to the app will not automatically get the Android Custom Tab browser to the foreground, but instead puts the app activity that handles the app-switch in the foreground.

If the intention is to have the Custom Tab to the foreground after the app-switch to the app, then it is possible to register an empty activity to handle the app-switch event from the app switch URL given to the MitID app flow and then let this activity close itself upon getting to the foreground based on this event. This will pop this activity and then get the Custom Tab to the foreground. Here it is required to register the Custom Tab as running in “single-task” mode.

## Detection of termination of Custom Tab flow
When the MitID flow completes after the app switch back from the MitID app to the integrating app, there is no inherent way to detect that the flow is completed and to automatically terminate the Custom Tab and return the flow into the app.

App schemes should be avoided and have pitfalls here, App Links will not automatically trigger at this last step, due to the last redirect in the flow not being based on user-interaction. The integrating service should in all scenarios implement the last redirect of the flow to render a button for the end-user which will then trigger an app switch back into the app.
Then, to avoid this last end-user button click, it is possible to setup a postMessage channel between the app and the Custom Tab instance, allowing the rendered page at the last step to notify the app of completion and pass along any required parameters as well, which will enable automatic termination of the flow.

## Custom Tab postMessage channel

When setting up the Custom Tab instance, the app can register a [postMessage channel](https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsSession#requestPostMessageChannel(android.net.Uri)), which allows for JavaScript running at allowed domains to communicate with the app via JS Post Messaging. 

This registration is done in similar ways as for App Link registration, by setting up appropriate settings in the assetlinks.json file in the root of the target domain.

This documentation explains how to integrate PostMessage communication between your Android app and your webpage hosted on **your-domain.com**. The guide covers:

- Configuring your Android app with a Custom Tab session callback listener.
- Registering a PostMessage channel with an optional secure challenge.
- Setting up your domain’s `assetlinks.json` for Digital Asset Links using `common.use_as_origin`.
- Writing JavaScript on your webpage to validate the custom scheme origin.

---

### 1.1. Setting Up the Custom Tab Session Callback

In your Android app, create a Custom Tab session with a callback listener that handles events related to the PostMessage channel. For example:

```java
// Import necessary packages
import android.os.Bundle;
import androidx.browser.customtabs.CustomTabsCallback;
import androidx.browser.customtabs.CustomTabsSession;
import androidx.browser.customtabs.CustomTabsIntent;
import android.net.Uri;

public class CustomTabHelper {

    private CustomTabsSession customTabsSession;

    // Initialize the session with a callback listener
    public void setupCustomTabSession() {
        CustomTabsCallback callback = new CustomTabsCallback() {
            @Override
            public void onMessageChannelReady(Bundle extras) {
                // PostMessage channel is ready. You can optionally send a challenge to the webpage.
                // Example: Send your challenge to the webpage for verification.
                String challenge = "your_generated_challenge"; // Optional: use this challenge for extra security.
                if (customTabsSession != null) {
                    customTabsSession.requestPostMessageChannel(Uri.parse("https://your-domain.com"));
                    customTabsSession.postMessage(challenge, null);
                }
            }

            @Override
            public void onPostMessage(String message, Bundle extras) {
                // Handle incoming messages from the webpage.
                // For example, you can log the message or trigger app logic.
                System.out.println("Received message: " + message);
            }
        };

        // Obtain a CustomTabsSession from your CustomTabsClient (binding code not shown)
        customTabsSession = /* Your code to create a session with callback */ null;
    }

    // Launch the Custom Tab with your domain URL
    public void launchCustomTab() {
        if (customTabsSession != null) {
            CustomTabsIntent intent = new CustomTabsIntent.Builder(customTabsSession).build();
            intent.launchUrl(/* context */, Uri.parse("https://your-domain.com"));
        }
    }
}
```
**Key Points:**
- onMessageChannelReady: Called when the PostMessage channel is ready. Optionally, your app sends a challenge string (which you should generate uniquely) to the webpage.
- onPostMessage: Receives messages from the webpage.
- The channel is registered for the domain https://your-domain.com.

#### Optional Secure Challenge
The challenge-response mechanism is optional. If used, your app sends a challenge (e.g., "your_generated_challenge") when the channel is ready. The webpage can then verify this challenge before processing further messages. If you choose not to implement this extra security measure, simply omit the challenge verification in both the app and the webpage code.

### Domain Association with Digital Asset Links
To allow your app to be associated with your webpage, host an assetlinks.json file on your domain under the path /.well-known/assetlinks.json.

Example assetlinks.json
```JSON
[
  {
    "relation": ["delegate_permission/common.use_as_origin"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.yourapp",
      "sha256_cert_fingerprints": [
        "AB:CD:EF:12:34:5..2:34:56:78:90"
      ]
    }
  }
]
```
**Notes:**

- package_name: Replace with your Android app’s package name.
- sha256_cert_fingerprints: Replace with your app’s certificate fingerprint.
- Host this file at https://your-domain.com/.well-known/assetlinks.json.

### Webpage JavaScript Integration
On your webpage hosted at your-domain.com, include JavaScript to listen for and validate messages coming from your Android app. The expected origin should follow the format: android-app://<your.app.domain>/<your.app.package>.

```JavaScript
// Listen for messages from the Android Custom Tab
window.addEventListener('message', function(event) {
  // Validate the origin: it should match the custom scheme for your Android app
  const validOrigin = "android-app://your.app.domain/your.app.package";
  if (event.origin !== validOrigin) {
    console.warn("Rejected message from invalid origin:", event.origin);
    return;
  }

  // Optionally, verify the challenge that your Android app sent.
  // If you are not using a challenge, you can skip this check.
  const expectedChallenge = "your_generated_challenge";
  if (event.data !== expectedChallenge) {
    console.error("Challenge verification failed.");
    return;
  }

  // Process the message if validation succeeds
  console.log("PostMessage received from Android app:", event.data);
});
```
**Key Points:**

- The listener checks that event.origin matches the expected Android app scheme.
- It verifies that the received message (challenge) matches what your app sent if you choose to use the challenge.
- Once verified, the page safely processes the message.

## Security Considerations
- Optional Challenge-Response: If implemented, generate unique challenge values per session and consider using a cryptographic random generator.
- Assetlinks: Ensure that your assetlinks.json is correctly configured to associate your domain with your Android app; any mismatch will prevent a secure connection.
- Origin Check: The JavaScript code must strictly verify the origin of incoming messages to avoid processing messages from unauthorized sources.

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

## Example assetlinks.json
Combining both App Links configuration and PostMessage handling, the resulting assetlinks.json will look something like

```JSON
{
  "relation": [
    "delegate_permission/common.handle_all_urls",
    "delegate_permission/common.use_as_origin"
  ],
  "target": {
    "namespace": "android_app",
    "package_name": "dk.your.app.production",
    "sha256_cert_fingerprints": [
      "46:61:19:30....2E:2E",
      "E6:0C:44.....4:53:D4"
    ]
  }
}
```

