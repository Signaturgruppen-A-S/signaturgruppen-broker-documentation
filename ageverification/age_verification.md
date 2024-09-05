---
title: Age Verification
layout: home
nav_order: 85
---

# Age Verification  

## Introduction

Ensuring the strong verification of an end-user's age online is crucial for many services. Signaturgruppen Broker offers a range of robust age verification flows that can be seamlessly integrated into almost any website or online service. This document outlines the technical options available for integrating strong age verification with MitID through Signaturgruppen Broker.

MitID provides secure authentication and identification, including the end-user's age and date of birth as standard claims in MitID flows. Services already integrated with MitID can leverage these claims for strong age verification. This document is particularly relevant for services without an active MitID integration or those seeking to use Signaturgruppen Broker's streamlined age verification flows.

Given that each MitID transaction incurs a cost, it is important to consider how frequently your service requests age verification in your workflows. For instance, a full MitID flow could be utilized during user registration, enabling the user to register their name, age, and birthday using MitID as a one-time operation. This allows frictionless age verification in future workflows. Alternatively, for workflows that prompt the end-user with an age verification step, a more dedicated MitID age verification flow may be desirable.

Signaturgruppen Broker supports two primary approaches to age verification:

1. **Standard MitID Integration**
2. **Specialized Age Verification Flows**

Services can utilize one or multiple variants dynamically. This document will outline and describe the technical integrations and example use cases for these options.

## Standard MitID Integration Support

A standard MitID integration with Signaturgruppen Broker is based on the OIDC interface. This integration results in a typical MitID flow for the end-user, returning the full set of MitID claims to the service, along with the associated integration requirements.

For an example, see our [online demo](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/#online-demo-of-signaturgruppen-broker).

## Specialized Age Verification Flows

Signaturgruppen Broker has implemented specific age verification flows that streamline the process for both services and end-users. These flows support OIDC integrations for both code authorization and implicit flows, as well as a pure JavaScript and REST API-based variant, enabling all types of services to integrate these flows.

These specialized flows ensure that the end-user is informed during the MitID flow about the context and what data is being returned to the service. The service only receives the actual age verification result, ensuring maximum user privacy, minimal data transfer, and no additional GDPR considerations.

Explore our [live demo in PP](https://brokerdemo-pp.signaturgruppen.dk/ageverify) (MitID test users) and our [live demo in production](http://brokerdemo.signaturgruppen.dk/ageverify) for an interactive demonstration.

We also offer a [demo example](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo) made with HTML and vanilla JavaScript, illustrating how an integration can be done without OpenID Connect or OAuth.

## Getting Started – Open Quick Testing

We provide two open test clients in our PP environment, along with MitID test users of different ages.

### MitID Test Users
> Password for all test users: '**AgeTest1234**'

- **age_verify_1985** (born in 1985)
- **age_verify_2008** (born in 2008)
- **age_verify_2011** (born in 2011)

### Test Clients
Two demo clients are available, both supporting any redirect_uri (not applicable for production integration).

> Note that the following examples are running standard OpenID Connect flows and is provided as a quick example of the integration. 
> It is highly recommended that a standard library supporting OpenID Connect is utilized, which is available for almost any platform.

**Code Authorization Example:**
```
client_id: 0b5ac04e-5dbb-4bb8-a697-93152896a2f6
client_secret: lk9JFkNkvodVPNA7E4Aui7oMal2HraIXwCqvOjEo1ymToAmsEdmLSPX8fevqvAaKSxmi1HHYufKnBOY9KCsLNQ==
Example URL: https://pp.netseidbroker.dk/op/connect/authorize?client_id=0b5ac04e-5dbb-4bb8-a697-93152896a2f6&redirect_uri=https://oidcdebugger.com/debug&response_type=code&scope=openid%20age_verify:16&prompt=login
```
The flow will return to **https://oidcdebugger.com/debug**, guiding you through the final step of retrieving the ID token from the Token endpoint using the **client_id** and **client_secret** provided.

For an introduction, please refer to our introduction guide to [Authorization Code Flow](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/openid-intro.html#authorization-code-flow---technical-steps).

**Implicit Flow Example:**
```
client_id: 9d3c7d79-96c4-43bc-8562-f0bf88ef69b8
Example URL: https://pp.netseidbroker.dk/op/connect/authorize?client_id=9d3c7d79-96c4-43bc-8562-f0bf88ef69b8&redirect_uri=https://oidcdebugger.com/debug&response_type=id_token&scope=openid%20age_verify:16&prompt=login&nonce=new-nonce
```
The Implicit flow returns a signed ID token directly to the specified redirect_uri, here **https://oidcdebugger.com/debug**, which displays the contents:

```
{
   ...
   "idbrokerdk_age_verified": "16:true"
}
```
The key claim in the ID token is **idbrokerdk_age_verified**, indicating the age verification result, formatted as 16:true/false.

## Age Verification Flows

Signaturgruppen Broker has introduced a streamlined flow for easy and efficient age verification. This ensures that only the specific age verification request is answered in the returned data, allowing for a data-minimized and GDPR-compliant integration.

### Key Features of These Flows:
- Available as both OpenID Connect Code Authorization and Implicit flow types.
- Supports a pure HTML and JavaScript integration.
- Streamlined and user-friendly.
- Does not require a client secret, making it suitable for public clients (embedded in apps).
- To support a strong age verification, only the validation of the resulting ID token and nonce requires a backend, the rest can be made entirely from the frontend.

To get started, use the provided example clients for the PP environment. When ready for production, contact Signaturgruppen to set up your integration with your own technical client.

## Age Verification with OpenID Connect

To integrate with OpenID Connect, include the following parameters in your request:

| Parameter      | Description  |
|----------------|--------------|
| client_id      | Client ID [UUID] |
| redirect_uri   | The https URL of the endpoint at the integrating service, which will receive the response redirect (`?code=xxxx&state_values=yyyy` for Authorization Code flow or `#id_token=zzzzz` for Implicit flow). |
| response_type  | 'code' for Authorization Code flow or 'id_token' for Implicit flow |
| scope          | Always set to 'openid age_verify:[age-to-verify]', e.g., 'openid age_verify:16' |
| nonce          | Unique and random, used to enable replay protection. Generated and verified by the integrating service, included in the resulting ID token. |
| prompt=login   | **Optional**: Forces authentication if specified. Without this, an active and usable user session at Signaturgruppen Broker will be used to allow for automatic reauthentication without user interaction. |

**Example Authorization Code Request:**
```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=0b5ac04e-5dbb-4bb8-a697-93152896a2f6&redirect_uri=https://oidcdebugger.com/debug&response_type=code&scope=openid%20age_verify:16&prompt=login
```

## Age Verification with HTML and JavaScript

Signaturgruppen Broker offers a JavaScript cross-document messaging API, allowing clients to handle the request and response in vanilla JavaScript without the need to manage OpenID Connect/OAuth redirects. An online demonstration is available [here](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo).

This integration is ideal for web applications that do not have OpenID Connect/OAuth integrations but can use JavaScript to handle the MitID Age Verification. The provided JavaScript example demonstrates how to set up a pop-up window starting an OIDC implicit flow, using a special Signaturgruppen Broker endpoint as the redirect_uri. This enables the ID token to be received via cross-document messaging and then verified using the [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html), specifically for the age verification scenario.

This approach allows almost any web application to integrate age verification without disrupting existing workflows.

### HTML and JavaScript - migrating example to production ready integration
The example HTML and JavaScript demo showcase the entire flow from start to finish implemented entirely in the frontend and provides a good starting point for the MitID Age Verification integration.
To enable a strong age verification, some parts of the flow should be moved to your backend. This section outlines the steps and considerations you should take to complete the integration.

**A quick outline of the steps to consider:**

1. Nonce generation and backend validation
2. ID token backend validation

**The overall flow consists of**

1. Generate nonce
2. Open popup and start flow using nonce
3. Receive reponse including ID token and close popup
4. Validate nonce and ID token in your backend, at any stage of your flow.

### Sequence diagram overview of Age Verification flow
Heres a simple overview of the interaction between your frontend, your backend and Signaturgruppen Broker. This is applicable for all our flow variants and highlights that the resulting ID token should be validated in your backend.

![Age Verify Frontend Backend](https://github.com/user-attachments/assets/bb760d49-5989-4453-bb0f-cee814ad93bd)

## Backend validation
When the flow completes successfully, an ID token is received from the JavaScript library provided.

In the HTML JavaScript demo example, this is validated directly online from JavaScript to demonstrate the steps needed. In the OpenID Connect Implicit flow, the ID token is received from a browser redirect and collected from your own JavaScript handler. 
In all variants and scenarios, the flow should end with a validation of the ID token in your backend. This section describes this validation and the considerations you should make when implementing this part for your integration.

> In short: For all variants (OIDC or HTML+JavaScript) the flow should end with a backend validation of the received ID token.

### ID Token Result

The ID token received from either the OpenID Connect flow types or the [JavaScript cross-document messaging example](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo) contains the following structure (example from PP):

```
{
   "iss": "https://pp.netseidbroker.dk/op",
   "nbf": 1725009225,
   "iat": 1725009225,
   "exp": 1725009525,
   "aud": "9d3c7d79-96c4-43bc-8562-f0bf88ef69b8",
   "nonce": "new-nonce",
   "sid": "2d08c82b-dd1f-441f-ae7d-5b690c4d23fa",
   "sub": "624256d3-4cac-44d1-8a97-0e967c015b6c",
   "auth_time": 1725009225,
   "idp": "idbrokerdk",
   "neb_sid": "2d08c82b-dd1f-441f-ae7d-5b690c4d23fa",
   "transaction_id": "90aceda9-da54-40aa-97ac-cd99eca38332",
   "idtoken_type": "idbroker.dk",
   "idbrokerdk_age_verified": "16:false"
}
```

This is a standard JWT ID token issued to the calling client_id (aud). The key claim for the Age Verification flow is **idbrokerdk_age_verified**, which contains the verified age and result in the "[age]:[true/false]" format. 

The **nonce** claim mirrors the nonce from the request and should be validated by the receiving service, providing replay protection and transaction linking. It is possible to generate the nonce in your frontend, as in the example demo, or it is possible to generate the nonce in your backend before the flow is initiated. It provides a mechanism to bind a flow to a specific ID token and should be utilized to ensure that the received ID token has never been seen before and optionally that the nonce matches the expected, if generated in the backend. A cache of seen nonces could be implemented, that ensures that within the timespan of accepted ID tokens (see **iat** claim: "issued at"), the same nonce is never accepted twice.

The **sub** claim is a request-specific random UUID, preventing user tracking across multiple requests.

The ID token is validated as a standard JWT (ID) token using the [JWKS endpoint of the Signaturgruppen Broker](https://pp.netseidbroker.dk/op/.well-known/openid-configuration/jwks) or by using the [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html). It is recommended to validate the ID token using the public verification key (see JWKS endpoint) cached or pinned, which prevents a binding to a HTTP call at this backend verification step. 

The [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html) allows verification of any valid ID token previously issued by Signaturgruppen Broker, regardless of the **exp** (expiration) claim in the ID token (normally 5 minutes from issuance). This allows this API to be utilized to verify the ID token at any point after it has been received. It is recommended that the integration service considers to utilize the **iat** (issued at) claim to setup their own expiration for how old they can be before validation.

#### Browser tampering
The backend steps are required to ensure that the user is unable to alter the age verification check in-browser. The resulting ID token is signed and thus any modification to this JWT token, will cause the validation to fail when done in the backend.
