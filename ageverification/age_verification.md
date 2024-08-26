# Signaturgruppen Age Verification

In order to strongly verify the age of an end-user online, Signaturgruppen Broker has implemented a range of possible age verification flows that allows almost any website or online service to integrate a strong age verification with MitID into their existing workflows. 
This document outlines the various technical options available for strong age verification when integrating to Signaturgruppen Broker.

## Introduction
MitID offers a strong autentification and identification and already includes the age and birthday of the end-user as standard MitID claims to the service provider for standard MitID flows. Thus, for services that already integrate to MitID the age and birthday claims returned could naturally be utilized as the basis for a strong age verification implementation.

This document is targeting services that does not already have an active MitID intgration or services that looks to utilize the more streamlined age verification flows supported by Signaturgruppen Broker. 

There is a cost associated with each started MitID transaction, so first thing to consider is how often your service is asking for MitID age verification in your workflows. As an example, your service could ask for a standard (full) MitID flow as part of a user registration process that would enable to user to register name, age and bithday using his or her MitID as a one time operation, that would allow frictionless age verification using this user registration for future workflows.
For workflows in which the end-user is prompted with an age verification step that has to be as frictionless as possible, a more dedicated age verification MitID flow might be required. 

To support these various workflows, Signaturgruppen Broker has implemented to overall approaches to Age Verification:

* Standard MitID integration support
* Specialized Age Verification flows

It is possible for services to utilize one or multiple variants dynamically. We will outline and describe the technical integrations and example usages in this document.

### Standard MitID integration support
A standard MitID integration is based on the Signaturgruppen Broker OIDC interface and results in a standard MitID flow for the end-user and the full set of end-user MitID claims is returned to the service, with all associated requirements for these integrations.

### Specialized Age Verification flows
Signaturgruppen Broker has implemented a couple of Age Verification specific flows that is more streamlined to the age verification process for both services and end-users. These flows supports OIDC integrations for both code authorization and implicit flow, as well as a pure JavaScript and REST API based variant that allows all types of services to integrate to these flows. 
In addition to being simple for services, the end-user will be informed as part of the MitID flow on the context and what data is returned to the service. 
The service only receives back the actual age verification result and nothing else, which enables maximum user privacy, minimal data transfer and no additional GDPR considerations for receiving services.
