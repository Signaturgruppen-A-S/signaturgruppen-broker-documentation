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

## Age Verification flows

* ID token returns age verification result.
* Response and tokens does only contain age verification response, nothing else.
* User is informed by the flow and inside the MitID client/app, exactly what is being approved and what data is handed out to the services.

### Example


### Age Verification with OpenID Connect
Signaturgruppen Broker has introduced a streamlined OpenID Connect flow that makes MitID age verification easily available and ensures that only the specific age verification request is answered in the returned data, allowing for a data-minimized and GDPR friendly integration. 
Notable for these flows:
* Available as both Code Authorization and Implicit flow types
* Does not require client secret, i.e. public clients (embedded in apps) are perfectly fine (see section on requirements and considerations).

### Age Verification with HTML and JavaScript
Signaturgruppen Broker has introduced a JavaScript cross-document messaging API, that allows integrating clients using the implicit flowtype to handle the request and response in vanilla JavaScript without any need to handle the redirect response from OpenID Connect/OAuth flows. An online demonstration has been published [here](https://github.com/Signaturgruppen-A-S/signaturgruppen-age-verification-demo).

The idea with integration is to allow easy integration from webapplications that does not otherwise have any OpenID Connect/OAuth integrations, but is able to utilize JavaScript to setup and handle the MitID Age Verification. 


## Requirements and considerations for a strong age verification
