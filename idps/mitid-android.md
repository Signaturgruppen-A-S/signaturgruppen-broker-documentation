---
title: Android MitID integration
layout: home
parent: App MitID Integration
grand_parent: MitID
grand_grand_parent: Identity providers
nav_order: 1
---

# Android MitID integration

On Android, the recommended approach is to use Custom Tabs targeting the Chrome browser. This method leverages the security and performance benefits of Chrome while allowing deep integration with your app.

Custom Tabs can be launched in one of two modes:

- **Single-instance mode:** The Custom Tab runs as a separate instance.
- **Single-task mode:** The Custom Tab becomes part of your app’s task.

In both cases, when returning to your app via App Links, the Custom Tab typically remains in the background. Although the browser can often complete the MitID flow in the background (provided the device isn’t in power-saving mode), there are situations where the Custom Tab must be brought to the foreground to ensure the flow completes successfully. It is also expected that the MitID client’s “finish” screen is visible to the user in the browser; therefore, it is generally advisable to keep the Custom Tab active until the flow is fully complete.

---

## Bringing the Custom Tab to the Foreground

A direct app switch back to your app does not automatically foreground the Custom Tab. Instead, the activity handling the app switch is displayed. To bring the Custom Tab to the front after the app switch, you can:

- **Register an Empty Activity:**  
  Create a minimal activity to handle the app-switch URL provided by the MitID flow. Once this activity receives the event, it immediately closes itself, which causes the Custom Tab to pop to the foreground.

- **Mode Requirement:**  
  This solution requires that the Custom Tab is launched in **single-task mode**.

---

## Detecting Termination of the Custom Tab Flow

When the MitID flow completes and the MitID app switches back to your app, there is no built-in mechanism to automatically detect that the flow is finished and to close the Custom Tab.

- **App Schemes are Not Ideal:**  
  Using custom app schemes for the final redirect is discouraged because they do not automatically trigger an app switch (the final redirect is not a user-initiated action) and carry inherent security risks.

- **Recommended Approach:**  
  The integrating service should implement the final redirect to render a button on the webpage. This button, when tapped by the end user, triggers the app switch back into your app.
  
- **Automating the Process:**  
  To avoid requiring the user to click a button, you can establish a postMessage channel between your app and the Custom Tab. This channel enables the webpage at the final step to automatically notify your app of the flow’s completion and pass any necessary parameters, thereby allowing for the automatic termination of the flow.

---

## Custom Tab postMessage Channel

Your Android app can register a [postMessage channel](https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsSession#requestPostMessageChannel(android.net.Uri)) when setting up the Custom Tab instance. This channel allows JavaScript running on allowed domains to communicate directly with your app via JS post messaging.

The registration process is similar to App Link configuration, relying on proper settings in the `assetlinks.json` file hosted at the root of your target domain.

This documentation explains how to integrate postMessage communication between your Android app and your webpage hosted on **your-domain.com**. The guide covers:

- Configuring your Android app with a Custom Tab session callback listener.
- Registering a postMessage channel with an optional secure challenge.
- Setting up your domain’s `assetlinks.json` for Digital Asset Links using `delegate_permission/common.use_as_origin`.
- Implementing JavaScript on your webpage to validate the custom scheme origin.


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

