# Signaturgruppen Age Verification

In order to strongly verify the age of an end-user online, Signaturgruppen Broker has implemented a range of possible age verification flows that allows almost any website or online service to integrate a strong age verification with MitID into their existing workflows. 
This document outlines the various technical options available for strong age verification when integrating to Signaturgruppen Broker.

## Introduction
MitID offers a strong autentification and identification and already includes the age and birthday of the end-user as standard MitID claims to the service provider for standard MitID flows. Thus, for services that already integrate to MitID the age and birthday claims returned could naturally be utilized as the basis for a strong age verification implementation.

This document is targeting services that does not already have an active MitID intgration or services that looks to utilize the more streamlined age verification flows supported by Signaturgruppen Broker. 

There is a cost associated with each started MitID transaction, so first thing to consider is how often your service is asking for MitID age verification in your workflows. As an example, your service could ask for a standard (full) MitID flow as part of a user registration process that would enable to user to register name, age and bithday using his or her MitID as a one time operation, that would allow frictionless age verification using this user registration for future workflows.
For workflows in which the end-user is prompted with an age verification step, a more dedicated age verification MitID flow might be desireable. 

To support these various workflows, Signaturgruppen Broker has implemented to overall approaches to Age Verification:

* Standard MitID integration support
* Specialized Age Verification flows

It is possible for services to utilize one or multiple variants dynamically. We will outline and describe the technical integrations and example usages in this document.

### Standard MitID integration support
A standard MitID integration is based on the Signaturgruppen Broker OIDC interface and results in a standard MitID flow for the end-user and the full set of end-user MitID claims is returned to the service, with all associated requirements for these integrations.

As an example, see our [online demo](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/#online-demo-of-signaturgruppen-broker)

### Specialized Age Verification flows
Signaturgruppen Broker has implemented a couple of Age Verification specific flows that is more streamlined to the age verification process for both services and end-users. These flows supports OIDC integrations for both code authorization and implicit flow, as well as a pure JavaScript and REST API based variant that allows all types of services to integrate to these flows. 

In addition to being simple for services, the end-user will be informed as part of the MitID flow on the context and what data is returned to the service. 
The service only receives back the actual age verification result and nothing else, which enables maximum user privacy, minimal data transfer and no additional GDPR considerations for receiving services.

See our [live demo in pp](https://brokerdemo-pp.signaturgruppen.dk/ageverify) (MitID testusers) and our [live demo in production](http://brokerdemo.signaturgruppen.dk/ageverify) for an interactive demonstration.

We have also published a [demo example made with HTML and vanilla JavaScript](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo), to illustrate how an integration can be done without OpenID Connect or OAuth.

## Getting started - open quick testing
Here we are introducing two open test clients usable in our PP environment and have included a set of MitID testusers with different ages.


### MitID Test users
> All have MitID password: '**AgeTest1234**'

MitID testusers
* **age_verify_1985** (born in 1985)
* **age_verify_2008** (born in 2008)
* **age_verify_2011** (born in 2011)

### Test clients
Two demo clients. 
Both supporting any redirect_uri (does not apply for production integration).

```
Code:
client_id: 0b5ac04e-5dbb-4bb8-a697-93152896a2f6
client_secret: lk9JFkNkvodVPNA7E4Aui7oMal2HraIXwCqvOjEo1ymToAmsEdmLSPX8fevqvAaKSxmi1HHYufKnBOY9KCsLNQ==
example url: https://pp.netseidbroker.dk/op/connect/authorize?client_id=0b5ac04e-5dbb-4bb8-a697-93152896a2f6&redirect_uri=https://oidcdebugger.com/debug&response_type=code&scope=openid%20age_verify:16&prompt=login
```
The flow will return at **https://oidcdebugger.com/debug**, which will guide you through the last step of retriving the ID token from the Token endpoint using the **client_id** and **client_secret** from above. 

```
Implicit (no secret)
client_id: 9d3c7d79-96c4-43bc-8562-f0bf88ef69b8
example url: https://pp.netseidbroker.dk/op/connect/authorize?client_id=9d3c7d79-96c4-43bc-8562-f0bf88ef69b8&redirect_uri=https://oidcdebugger.com/debug&response_type=id_token&scope=openid%20age_verify:16&prompt=login&nonce=new-nonce
```

The Implicit flows returns a signed ID token directly back to the redirect_uri specified in the request, here **https://oidcdebugger.com/debug**,  which will show you the contents of this, examplified here:

```
{
   ...
   "idbrokerdk_age_verified": "16:true"
}
```
The important claim from the ID token, is the **idbrokerdk_age_verified** which has the value of the age verified and the verification result, 16:true/false for this demonstration. 

## Age Verification flows

* ID token returns age verification result.
* Response and tokens does only contain age verification response, nothing else.
* User is informed by the flow and inside the MitID client/app, exactly what is being approved and what data is handed out to the services.

### Age Verification with OpenID Connect
Signaturgruppen Broker has introduced a streamlined OpenID Connect flow that makes MitID age verification easily available and ensures that only the specific age verification request is answered in the returned data, allowing for a data-minimized and GDPR friendly integration. 
Notable for these flows:
* Available as both Code Authorization and Implicit flow types
* Does not require client secret, i.e. public clients (embedded in apps) are perfectly fine (see section on requirements and considerations).

### Age Verification with HTML and JavaScript
Signaturgruppen Broker has introduced a JavaScript cross-document messaging API, that allows integrating clients using the implicit flowtype to handle the request and response in vanilla JavaScript without any need to handle the redirect response from OpenID Connect/OAuth flows. An online demonstration has been published [here](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo).

The idea with integration is to allow easy integration from webapplications that does not otherwise have any OpenID Connect/OAuth integrations, but is able to utilize JavaScript to setup and handle the MitID Age Verification. 

## ID token result
The ID token received from either the OpenID Connect flowtypes or the [JavaScript cross-document messaging example](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo), will contain the following datastructure (example from PP)

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
This is a standard JWT ID token, which is issued towards the calling client_id (aud). The important claim of note for the Age Verification flow, is the **idbrokerdk_age_verified** claim, which contain the age verified and the verification result in the "[age]:[true/false]" format.
The **nonce** claim will mirror the provided nonce from the starting request, and should be validated by the receiving service.
The **sub** claim is a pr. request random UUID and cannot be used to track the user across multiple requests.

This ID token can validated as a standard OpenID Connect/OAuth ID token using the discovery endpoint of the Signaturgruppen Broker or by using the [Token Validation API endpoint](https://pp.netseidbroker.dk/op/swagger/index.html).

> It is strongly recommended that the nonce generation and validation of nonce and ID token is done in a backend implementation. It is possible, as demonstrated in our HTML+JS demo, to setup the integration entirely in-browser and not utilize any backend for this, but this exposes the 
> validation to tampering in-browser by users who want to circumvent this validation mechanism. Signaturgruppen has provided the means to integrate a strong age verification into any web or app context, it is ultimately up to the integrating service to secure their integration.
