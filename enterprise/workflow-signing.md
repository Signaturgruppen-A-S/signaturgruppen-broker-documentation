# Workflow Signing

## Introduction
Signaturgruppen Broker Workflow signing is an API and OpenID Connect workflow that supports advanced and highly-flexible scenarios of PDF document signing with one or more documents and one or more signers.
It has been designed to support any supported identity provider and eID scheme supported by Signaturgruppen Broker and allows for customization of the flow, number of documents and signers. 
The workflow is dynamic and supports adding additional signers and various ways to get the final result of the flow, depending on requirements and needs.

When a workflow has been completed, the result can be delivered in both a sealed PAdES signed with a EU trustlist signing service and a more compact JWT format, both supporting various storage and data requirements.

## Online demo
We have setup a working online demonstration at [https://brokerdemo-pp.signaturgruppen.dk/workflows](https://brokerdemo-pp.signaturgruppen.dk/workflows), which provides a simple visual representation of the core features of Workflow Signing.

### MitID test identities
We have setup the demo to utilized MitID PP test identities and setup the demo to use [Signaturgruppen Broker Simulation](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/enterprise/simulation.html) for easier and faster testing. 

You can utilize any MitID PP test user, or use the premade users listed here:

| MitID PP username |
|--------|
| workflow_signing_demo_1       |
| workflow_signing_demo_2       |
| workflow_signing_demo_3       |

## Swagger
The swagger reference for Workflow API (PP env) is found at: https://pp.netseidbroker.dk/transactionsigning/api/swagger/index.html?urls.primaryName=Workflow+API
