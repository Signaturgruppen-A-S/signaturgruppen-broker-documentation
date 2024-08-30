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

**Code Authorization Example:**
```
client_id: 0b5ac04e-5dbb-4bb8-a697-93152896a2f6
client_secret: lk9JFkNkvodVPNA7E4Aui7oMal2HraIXwCqvOjEo1ymToAmsEdmLSPX8fevqvAaKSxmi1HHYufKnBOY9KCsLNQ==
Example URL: https://pp.netseidbroker.dk/op/connect/authorize?client_id=0b5ac04e-5dbb-4bb8-a697-93152896a2f6&redirect_uri=https://oidcdebugger.com/debug&response_type=code&scope=openid%20age_verify:16&prompt=login
```
The flow will return to **https://oidcdebugger.com/debug**, guiding you through the final step of retrieving the ID token from the Token endpoint using the **client_id** and **client_secret** provided.

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
- A backend is recommended but not technically required.

To get started, use the provided example clients for the PP environment. When ready for production, contact Signaturgruppen to set up your integration with your own technical client.

### Age Verification with OpenID Connect

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

### Age Verification with HTML and JavaScript

Signaturgruppen Broker offers a JavaScript cross-document messaging API, allowing clients to handle the request and response in vanilla JavaScript without the need to manage OpenID Connect/OAuth redirects. An online demonstration is available [here](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo).

This integration is ideal for web applications that do not have OpenID Connect/OAuth integrations but can use JavaScript to handle the MitID Age Verification. The provided JavaScript example demonstrates how to set up a pop-up window starting an OIDC implicit flow, using a special Signaturgruppen Broker endpoint as the redirect_uri. This enables the ID token to be received via cross-document messaging and then verified using the [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html), specifically for the age verification scenario.

This approach allows almost any web application to integrate age verification without disrupting existing workflows.

## ID Token Result

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
This is a standard JWT ID token issued to the calling client_id (aud). The key claim for the Age Verification flow is **idbrokerdk_age_verified**, which contains the verified age and result in the "[age]:[true/false]" format. The **nonce** claim mirrors the nonce from the request and should be validated by the receiving service, providing replay protection and transaction linking. The **sub** claim is a request-specific random UUID, preventing user tracking across multiple requests.

This ID token can be validated as a standard OpenID Connect/OAuth ID token using the discovery endpoint of the Signaturgruppen Broker or by using the [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html).

> While it's technically possible to handle the integration entirely in-browser, it is strongly recommended that nonce generation and ID token validation occur on a backend to secure the integration against tampering.
