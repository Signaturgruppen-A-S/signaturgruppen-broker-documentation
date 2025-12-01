# Workflow Signing

## Introduction
Signaturgruppen Broker Workflow signing is an API and OpenID Connect workflow that supports advanced and highly-flexible scenarios of PDF document signing with one or more documents and one or more signers.
It has been designed to support any supported identity provider and eID scheme supported by Signaturgruppen Broker and allows for customization of the flow, number of documents and signers. 
The workflow is dynamic and supports adding additional signers and various ways to get the final result of the flow, depending on requirements and needs.

When a workflow has been completed, the result can be delivered in both a sealed PAdES signed with a EU trustlist signing service and a more compact JWT format, both supporting various storage and data requirements.

## Resources

### Online demo
We have setup a working online demonstration at [https://brokerdemo-pp.signaturgruppen.dk/workflows](https://brokerdemo-pp.signaturgruppen.dk/workflows), which provides a simple visual representation of the core features of Workflow Signing.

### MitID test identities
We have setup the demo to utilized MitID PP test identities and setup the demo to use [Signaturgruppen Broker Simulation](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/enterprise/simulation.html) for easier and faster testing. 

You can utilize any MitID PP test user, or use the premade users listed here:

| MitID PP username |
|--------|
| workflow_signing_demo_1       |
| workflow_signing_demo_2       |
| workflow_signing_demo_3       |

### Swagger
The swagger reference for Workflow API (PP env) is found at: https://pp.netseidbroker.dk/transactionsigning/api/swagger/index.html?urls.primaryName=Workflow+API

## Technical overview
Workflow Signing consist of an API and an OpenID Connect interface, that together allow for the creation of workflows that supports PDF document signing for one or more signers, decoupled over time with various requirements and restrictions.

It starts with the creation of a workflow by uploading one or more PDF documents for signing and specified relevant parameters for the specific flow, like workflow title, controlling UI experience etc.

Anytime a new signer is ready to sign, a signtext ID is retrieved from the API, which is used in the OpenID Connect integration towards Signaturgruppen Broker. 

Upon successful signature, the integrating system has full control over what happened and by whom and is able to retrieve the result in both PAdES and a signed JWT for long-term storage formats.

The integrating service has full control over signers, can add more dynamically and is able to retrieve the resulting documents and tokens at any time.

### Visual walkthrough


## Example workflow

### Create workflow
A workflow is creating by uploading 
```
POST /api/workflows
{
  "title": "Workflow test title",
  "pdfList": [
    {
      "title": "Agreement 1",
      "pdfBase64": "..."
    },
    {
      "title": "",
      "pdfBase64": "..."
    }
  ],
  "expiresAt": "2025-12-01T11:48:48.900Z",
  "showStartOverviewPage": true
}
```
A response is given with the created workflow ID:
```
{
  "workflowId": "workflowid1234"
}
```
### Initiate signature
In this scenario we have an end-user and you want this end-user to go through the workflow created in the last step.
For this, the backend must retrieve a **signtext ID**, which is then used as OIDC parameter for the desired flow through Signaturgruppen Broker

Retrieve **signtext ID**: 
```
POST /api/workflows/signtextid
{
  "workflowId": "workflowid1234"
}
```



