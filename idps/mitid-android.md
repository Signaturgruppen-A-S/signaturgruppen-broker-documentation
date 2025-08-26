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

When returning to your app via App Links, the Custom Tab typically remains in the background. Although the browser can often complete the MitID flow in the background (provided the device isn’t in power-saving mode), there are situations where the Custom Tab must be brought to the foreground to ensure the flow completes successfully. It is also expected that the MitID client’s “finish” screen is visible to the user in the browser; therefore, it is generally advisable to keep the Custom Tab active until the flow is fully complete.

---

## Recommended Integration Strategy for Android

To ensure the most robust and user-friendly integration with MitID on Android, the following strategy is recommended:

### 1. Prefer Chrome for Custom Tabs
Use a **Custom Tab** for launching the MitID Broker flow, and **"pin" the Custom Tab to Chrome** if it is installed and available on the user's device. Chrome offers the most consistent and reliable support for the required features, including the `postMessage` communication protocol. If Chrome is not available, gracefully **fall back to the system default browser** to maintain compatibility.

### 2. Use `postMessage` Protocol When Available
Leverage the `postMessage` protocol for communication between the browser and your app when it is supported by the environment. This enables a **seamless and secure** return flow from the browser back to your app without requiring explicit user interaction.

- When supported, the `postMessage` mechanism should be the **default method** for handling the return to your app.
- Ensure your app listens for the relevant messages securely and within the appropriate browser context.

### 3. Fallback to App Switch Redirect with optional push
If the `postMessage` flow is not supported (e.g., due to browser limitations or device settings), implement a fallback using an **HTML "app switch redirect button"** presented to the user. This button should clearly instruct the user to return to your app by triggering the app switch manually.

- Optionally, if your system supports it and the user has enabled the feature, you may also use a **backend-initiated  data push message** to the app. This push can trigger the switch from the browser back to the app, offering a smoother fallback experience.

This dual fallback mechanism helps to ensure that users can complete the MitID authentication reliably, even when automatic redirection is not possible.

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
  To avoid requiring the user to click a button, you can establish a postMessage channel between your app and the Custom Tab, and optionally use data push messages when available. This channel enables the webpage at the final step to automatically notify your app of the flow’s completion and pass any necessary parameters, thereby allowing for the automatic termination of the flow.

---

## Custom Tab postMessage Channel

Your Android app can register a [postMessage channel](https://developer.android.com/reference/androidx/browser/customtabs/CustomTabsSession#requestPostMessageChannel(android.net.Uri)) when setting up the Custom Tab instance. This channel allows JavaScript running on allowed domains to communicate directly with your app via JS post messaging.

The registration process is similar to App Link configuration, relying on proper settings in the `assetlinks.json` file hosted at the root of your target domain.

This documentation explains how to integrate postMessage communication between your Android app and your webpage hosted on **your.app.domain**. The guide covers:

- Configuring your Android app with a Custom Tab session callback listener.
- Registering a postMessage channel with an optional secure challenge.
- Setting up your domain’s `assetlinks.json` for Digital Asset Links using `delegate_permission/common.use_as_origin`.
- Implementing JavaScript on your webpage to validate the custom scheme origin.


### 1.1. Setting Up the Custom Tab Session Callback

In your Android app, create a Custom Tab session with a callback listener that handles events related to the PostMessage channel. For example:

```java
...
var uri = "your-origin".toUri(); //ex https://your-domain
private val customTabsCallback =
    object : CustomTabsCallback() {
        override fun onNavigationEvent(navigationEvent: Int, extras: Bundle?) {
            super.onNavigationEvent(navigationEvent, extras)
            if (validated && navigationEvent == NAVIGATION_FINISHED) {
            //After navigation, postMessageChannel needs to be recreated
                  session?.requestPostMessageChannel(
                      uri, uri, bundleOf()
                  )
            }
        }

        override fun onPostMessage(message: String, extras: Bundle?) {
            super.onPostMessage(message, extras)
            //Validate the message, i.e. if you use this protocol to hand OIDC code to the app.
            //Ignore messages that does not conform to expectations - in principle you can receive unexpected messages here
            //As an example, you might have sent a challenge in onMessageChannelReady or via your backend to the web application and expect this challenge returned here
            //If challenge mechanism used, expect replay of challenge here


            //Here a new activity is startet, which will terminate the CustomTab and get focus back to the app.
            context?.startActivity(Intent(context, MainActivity::class.java))
        }
        
        override fun onMessageChannelReady(extras: Bundle?) {
            String challenge = "your_generated_challenge"; // Optional
            super.onMessageChannelReady(extras)
            session?.postMessage(challenge, null)
        }

        override fun onRelationshipValidationResult(
            relation: Int,
            requestedOrigin: Uri,
            result: Boolean,
            extras: Bundle?
        ) {
            super.onRelationshipValidationResult(relation, requestedOrigin, result, extras)
            validated = result
            //initiates the postMessageChannel with the intial page in-browser, optional if you have to wait for navigation events
            if (validated) {
                session?.requestPostMessageChannel(
                    uri, uri, bundleOf()
                )
            }
            if(!validated){
              //consider to handle the event that the validation is not successful - log to your app-backend, catch when debugging etc.
              //See the section below: "Domain Association with Digital Asset Links"
            }
        }
    }
```

```java
private fun bindCustomTabsService(url: String) {
    val packageName = CustomTabsClient.getPackageName(requireContext(), null)

    CustomTabsClient.bindCustomTabsService(
        requireContext(),
        packageName,
        object : CustomTabsServiceConnection() {
            override fun onCustomTabsServiceConnected(
                name: ComponentName,
                client: CustomTabsClient
            ) {
                this@FlowFragment.client = client
                client.warmup(0L)
                session = this@FlowFragment.client?.newSession(customTabsCallback)
                launch(url)

                //Initial validation of RELATION_USE_AS_ORIGIN, which ensures that the uri (source origin) has properly setup for postMessages for the ORIGIN (uri) 
                session?.validateRelationship(RELATION_USE_AS_ORIGIN, uri, null)
            }

            override fun onServiceDisconnected(componentName: ComponentName) {
                client = null
            }
        })
}

private fun launch(url: String) {
    val customTabsIntent = CustomTabsIntent.Builder(session!!)
        .build()
    customTabsIntent.launchUrl(requireContext(), url.toUri())
}
```

**Key Points:**
- session?.validateRelationship: Validates the the origin is properly setup.
- onNavigationEvent: After each navigation, setup the channel again.
- onMessageChannelReady: Called when the PostMessage channel is ready.
- onPostMessage: Receives messages from the webpage.
- The channel is registered for the domain https://your.app.domain.

#### Optional Secure Challenge
The challenge-response mechanism is optional. If used, your app sends a challenge (e.g., "your_generated_challenge") when the channel is ready and when the app receives an event from the webpage the same challenge can be verified. If you choose not to implement this extra security measure, simply omit the challenge verification in both the app and the webpage code.

This mechanism ensures that received PostMessage events in the app comes from the initial CustomTab session PostMessage channel.

### Domain Association with Digital Asset Links
To allow your app to be associated with your webpage, host an assetlinks.json file on your domain under the path /.well-known/assetlinks.json.

Example assetlinks.json
```json
[
  {
    "relation": ["delegate_permission/common.use_as_origin"],
    "target": {
      "namespace": "android_app",
      "package_name": "your.app.package",
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
- Host this file at https://your.app.domain/.well-known/assetlinks.json.

### Webpage JavaScript Integration
On your webpage hosted at your.app.domain, include JavaScript to listen for and validate messages coming from your Android app. The expected origin should follow the format: android-app://<your.app.domain>/<your.app.package>.

```javascript

function exampleGenerateResponseMessage(_) {
    var msg = document.getElementById('postMessageResult');
    if (msg) {
        return msg.textContent;
    }
    return null;
}

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

  var port = event.ports[0];
  if (typeof port === 'undefined') {
    console.log("Port is undefined, ignoring..");
    return;
  }

  try
  {
    var msg = exampleGenerateResponseMessage(event);
    if (msg) {
        console.log("Sending postMessage result back..");
        port.postMessage(msg);
    } else {
        console.log("No postMessage result to send back..");
        port.postMessage("N/A");
    }
  } catch (e) {
    console.log("Error: " + e);
  }
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

```json
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

