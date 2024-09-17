---
title: MitID iframe
layout: home
parent: MitID
grand_parent: Identity providers
nav_order: 3
---

## MitID Iframe model

MitID supports a restricted iframe model that allows to complete the MitID flow within an iframe as alternative to the normal full-page or pop-up OpenID Connect flows.
The MitID iframe variant, is available as **MitID reauthentication** only, which requires the initiating service provider to have some form of authentication with the end-user when starting the MitID and is required to provide either the MitID UUID or the Danish CPR.

This model has several restrictions that includes

- Only the MitID app and code-display authenticators are available.
- Must be initialized with end-user MitID UUID or Danish CPR.
- Still requires MitID accepted browsers, the same list as for normal MitID, i.e. **embedded browsers are not allowed**.

But in return has some advantages as well

- Username input step skipped.
- MitID app-switch can be made automatic.
- No redirect or pop-up required.

The iframe variant is generally available to service providers, but can only be used when the service provider has an active authentication of the end-user, i.e. already knows who the user is before starting the flow. This means, that the flow must not be started solely on the MitID UUID or CPR from the end-user without any form of authentication and validation of this. As an example, you could have some authentication with a local user from which you have a preregistered MitID UUID in your database, which then can be used to initiate the MitID reauthentication flow.

## Signaturgruppen Broker MitID iframe protocol

Overall, the flow will be handled by server-to-server API calls for initializing the flow and retrieving the signed ID token as well as setting up an iframe and communicating with Signaturgruppen Broker through the iframe using cross-document messaging.

The following sequence diagram illustrates the overall flow.

- Service invokes form post API endpoint with requested flow parameters
- Service retrieves auth_token
- Iframe and cross-document messaging is setup using auth_token, flow initiates in iframe
- An auth_code is returned via cross-document messaging to the end-user browser
- Service exchanges auth_code using secure client (secret/keys) from the Signaturgruppen Broker token endpoint.
- Service retrieves ID token. Validate ID token, see [Security section](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/tech-security.html)

## Security considerations

- The auth_code has a short lifetime and can only be used once.
- (Optional, but recommended): When retrieving ID token with auth_code the nonce in the ID token must match the nonce used in the flow parameters.
- ID token is constructed as for other Signaturgruppen Broker MitID flows, signed by the same HSM keys. The signing key is found in documentation and on the discovery endpoint.

## Client setup
In order to integrate to the MitID iframe setup, you first need a valid client. Start with the example PP client included in the examples on this page and then reach out to Signaturgruppen when you are ready to move on to your own integration. 

A client needs a **client secret** and a valid **redirect_uri**. If you are setting up the client yourself in our Admin UI, then ensure that the client has
* Require client secret and atleast one client secret
* One or more valid redirect_uri
* Setup for the **code** (Authorization Code flow)

## Flow overview

### 1) Initialize authentication flow

First step is to initialize the iframe authentication flow. This is done by invoking the following endpoint with a form POST (form url encoded).

```
POST [AuthorityUrl]/api/v1/iframe/initialize
```

This endpoint returns an authentication token used to initialize an authentication via the iframe setup.

The endpoint is invoked in the same manner as the Token endpoint using client secret, supporting the same formats for client info and client secret as the standard Token endpoint (OIDC and OAuth).

Additionally, information about the requested information is required, which is in the same format as a normal OpenID Connect request for the specified identity provider as for the standard OIDC flows through Signaturgruppen Broker. See **\[NEB-INTRO\]** , **\[NEB-TECHREF\]**  and **\[NEB-IDP\]** for reference.

To control the available MitID authenticators, set the MitID specific parameter “authenticators”, to “app” for app-only and to “app code_token” to support both app and code token display.

It is required to allow the end-user to app-switch to the MitID app from the iframe, this is done by allowing/not restricting the opening of new pages from user-clicking in the iframe sandboxing parameters.

To enable the “minimized” mode, which better adapts the MitID client inside the iframe to small screen sizes, set the MitID specific parameter “iframe_mode” to "minimized".

### 2) Iframe flow

The iframe flow is started by setting up an iframe using the iframe_url from the initialize response and then providing the auth_token as messaging parameter, as described below.

_The current url for pp for MitID flows is:_ [_https://netseidbroker.pp.mitid.dk/iframe_](https://netseidbroker.pp.mitid.dk/iframe) _(subject to change, follow the “iframe_url” parameter)._

Messages sent from Signaturgruppen Broker to the surrounding parent will be communicated in a JSON structure always including a command property used as an event type.

#### Iframe cross-document protocol steps
When an event is received, the **event.data** will contain the **command** and **message** parameters.

See the example JS handler here, which illustrates the steps to take in order to start the flow and receive the response using JavaScript cross-origin post messaging.
In order to initiate an iframe flow, you will need to provide a valid **auth_token** as response to the **ready** command. When the flow completes, a **authorization_code** or **error** response will be sent back.

```
<script>
    var handler = function(event) {
        if (event.data.command === 'ready') {
            var result = [];
            result["command"] = "auth_token";
            result["message"] = "<AUTH TOKEN>"; // setup your HTML to include the unique auth_code received from the initialize call
            event.source.postMessage(result, "*");
            return;
        };

        if (event.data.command === 'authorization_code') {
            var auth_code = event.data.message;
            handleSuccess(auth_code); //implement this
        }

        if (event.data.command === 'error') {
            var error = event.data.message;
            handleError(error); //implement this
        }
    };

window.addEventListener('message', handler); 
</script>
```

### 3) Token endpoint and tokens

## Initialization parameters

### Flow parameters
<table>
   <tbody>
      <tr>
         <th>
            <p><strong>Flow parameters</strong></p>
         </th>
         <th>
            <p><strong>Description</strong></p>
         </th>
         <th>
            <p><strong>Required/Optional</strong></p>
         </th>
      </tr>
      <tr>
         <td>
            <p>client_id</p>
         </td>
         <td>
            <p>Client ID</p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
      <tr>
         <td>
            <p>client_secret</p>
         </td>
         <td>
            <p>Client secret</p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>response_type</p>
         </td>
         <td>
            <p>Response type. Always use: <strong>code</strong></p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>scope</p>
         </td>
         <td>
            <p>Scopes requested. Must include <strong>openid</strong></p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>idp_values</p>
         </td>
         <td>
            <p>Always set to <strong>mitid</strong></p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>redirect_uri</p>
         </td>
         <td>
            <p>One of the valid redirect uris registered for the client</p>
            <p>The same redirect uri must be used when exchanging the received code for tokens</p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>idp_params</p>
         </td>
         <td>
            <p>MitID IdP parameters</p>
            <p>See next section for specification</p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>nonce</p>
         </td>
         <td>
            <p>Nonce (number once)</p>
            <p>Use a unique and hard to guess identifier for each flow. Echoed in the resulting ID token.</p>
         </td>
         <td>
            <p>Optional, but highly recommended</p>
         </td>
      </tr>
    <tr>
         <td>
            <p>language</p>
         </td>
         <td>
            <p>One of</p>
            <p>Danish: <strong>da</strong>, english: <strong>en</strong> or greenlandic: <strong>gl</strong></p>
            <p>Deafults to <strong>da</strong></p>
         </td>
         <td>
            <p>Optional</p>
         </td>
      </tr>
       <tr>
         <td>
            <p>iframe_mode</p>
             <p><strong>Requires special permission from MitID, contact Signaturgruppen if required</strong></p>
         </td>
         <td>
            <p><strong>minimized</strong></p>
             <p>Enables MitID minimized UI mode.</p>
         </td>
         <td>
            <p>Optional</p>
         </td>
      </tr>
   </tbody>
</table>

When the iframe flow has finished an auth_token will be communicated back via the cross-document messaging which can be used in the standard way to exchange for tokens via the Signaturgruppen Broker token endpoint.

### MitID IdP parameters (idp_params)
<table>
   <tbody>
      <tr>
         <th>
            <p><strong>IdP parameter</strong></p>
         </th>
         <th>
            <p><strong>Description</strong></p>
         </th>
         <th>
            <p><strong>Required/Optional</strong></p>
         </th>
      </tr>
      <tr>
         <td>
            <p>cpr_hint</p>
         </td>
         <td>
            <p>Danish CPR of user</p>
         </td>
         <td>
            <p>One of cpr_hint or uuid_hint is required</p>
         </td>
      </tr>
     <tr>
         <td>
            <p>uuid_hint</p>
         </td>
         <td>
            <p>MitID UUID of user</p>
         </td>
         <td>
            <p>One of cpr_hint or uuid_hint is required</p>
         </td>
      </tr>
      <tr>
         <td>
            <p>authenticators</p>
             <p><strong>Two authenticators are available: MitID App and MitID Code Token</strong></p>
         </td>
         <td>
            <p>One of</p>
             <p>Single string containing one or both of</p>
             <p><strong>app</strong>, <strong>code_token</strong></p>
             <p>Example: "app code_token"</p>
         </td>
         <td>
            <p>Required</p>
         </td>
      </tr>
    
   </tbody>
</table>

## Example flow for PP

Use the appropriate MitID UUID or CPR for a valid MitID PP testuser in the following example. 

```
curl --location 'https://pp.netseidbroker.dk/op/api/v1/iframe/initialize' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_secret=3PzE2SeZ5INHx7TCVyY7KQr1WSozw7ADytm1suAZ/UK1MCh9ZYQGdzN1BEv6hdzRsVn3xnA0/F/6ET9j0mTWWw==' \
--data-urlencode 'client_id=c0cb708e-f742-46a4-b664-9e31e32b145d' \
--data-urlencode 'response_type=code' \
--data-urlencode 'scope=openid mitid' \
--data-urlencode 'idp_values=mitid' \
--data-urlencode 'redirect_uri=https://oidcdebugger.com/debug' \
--data-urlencode 'idp_params={"mitid": {"cpr_hint": "[CPR]", "iframe_mode": "minimized", "authenticators": "app code_token"}}'
```

**Example success response:**

```
{
    "auth_token": "a1d...a0",
    "auth_token_expiration": 1726564561,
    "iframe_url": "https://netseidbroker.pp.mitid.dk/iframe"
}
```

**Error response is one of:**
* Not authorized
* Missing parameter
* Invalid redirect uri
  
## Iframe demo
A minimalistic iframe integration demo can be [found here](https://github.com/Signaturgruppen-A-S/mitid-iframe-quick-demo)
