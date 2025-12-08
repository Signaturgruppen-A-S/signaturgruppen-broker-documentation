---
title: Workflow Signing
layout: home
parent: Enterprise features
has_children: true
nav_order: 10
---

# Workflow Signing
**Workflow signing is in the final stage of development and has not yet been released for production. 
Several additional features are under development. 
Documentation (this), is not completed, but we invite integration and testing in our PP environment and welcome feedback. 
We plan to release Workflow Signing to production at latest January 2026.**

## Introduction
Signaturgruppen Broker Workflow signing is an API and OpenID Connect workflow that supports advanced and highly-flexible scenarios of PDF document signing with one or more documents and one or more signers.
It has been designed to support any supported identity provider and eID scheme supported by Signaturgruppen Broker and allows for customization of the flow, number of documents and signers. 
The workflow is dynamic and supports adding additional signers and various ways to get the final result of the flow, depending on requirements and needs.

**The Workflow API is designed with the intent to be a minimal but full fledged API that can serve as the foundation for both full-scale signature portals to simple signature flows - Giving the integrating service full control and flexibility to create their own signing flows.**

When a workflow has been completed, the result can be delivered in both a sealed PAdES signed with a EU trustlist signing service and a more compact JWT format, both supporting various storage and data requirements.

## Resources

### Online demo
We have setup a working online demonstration at [https://brokerdemo-pp.signaturgruppen.dk/workflows](https://brokerdemo-pp.signaturgruppen.dk/workflows), which provides a simple visual representation of the core features of Workflow Signing.
The demo provides a mixed-usecase toolbox, that allows for the creation of workflows using pre-configured PDFs or uploading your own and allows for adding signatures, see the result and fetch the resulting PAdES. 

### MitID test identities
We have setup the demo to utilize MitID PP test identities paired with [Signaturgruppen Broker Simulation](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/enterprise/simulation.html) for easier and faster testing. 

> Signaturgruppen Simulation is an enterprise feature that enables faster and more streamlined testing with simulated identities, like MitID and MitID Erhverv. Works even when MitID or MitID Erhverv is down and is highly recommended for end-2-end testing purposes.

You can utilize any MitID PP test user, or use the premade users listed here:

| MitID PP username |
|--------|
| workflow_signing_demo_1       |
| workflow_signing_demo_2       |
| workflow_signing_demo_3       |

### Swagger and Client Credentials (API client)
The swagger reference for Workflow API (PP env) is found at: [https://pp.netseidbroker.dk/transactionsigning/api/swagger/index.html?urls.primaryName=Workflow+API](https://pp.netseidbroker.dk/transactionsigning/api/swagger/index.html?urls.primaryName=Workflow+API)

In order to interact with the API, you must first get an [API client and retrieve a service token](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/api-integration.html). When retriving the service token from the Token endpoint, specify the **signtext_api** scope.

Contact Signaturgruppen Support (support.signaturgruppen@ingroupe.com) if you need to get up and running for our PP environment.

#### Example Client Credentials request:
```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'scope=signtext_api' \
--data-urlencode 'client_id=[client ID]' \
--data-urlencode 'client_secret=[client secret]'
```

#### Public test client
You can use the following API client to test the integration. The client has been created under the fictive CVR: DK00000002. 

>Note, that using this will share the workflows created with other integrations testing with this.

**ClientID:**
```
be3d42d0-1cc9-44df-b59c-3baeda398612
```
**Client Secret:**
```
xjqMzDt1721xhEsw8yufqBbwdLm6iJtq5Iafehy10bJ3zA4h1VqP8/OwNHfHMxoWy91aZZI1jAzeHv0NTn4gQA==
```

## Technical overview
Workflow Signing consist of an API and setting one additional parameter for the Signaturgruppen Broker OpenID Connect interface; this together allow for the creation of workflows that supports PDF document signing for one or more signers, decoupled over time with various requirements and restrictions.

It starts with the creation of a workflow by uploading one or more PDF documents for signing together with relevant parameters for the specific flow like workflow title and UI experience parameters.

Anytime a new signer is ready to sign, a signtext ID is retrieved from the API, which is used in the OpenID Connect integration towards Signaturgruppen Broker. 

Upon successful signature, the integrating system has full insight into what happened and by whom and is able to retrieve the results in both PAdES and a signed JWT for long-term storage formats.

The integrating service has full control over signers, can add more dynamically and is able to retrieve the resulting documents and tokens at any time.

### Authentication, approval and OpenID Connect
The Workflow Signing flows works as part of the Signaturgruppen Broker platform and has been integrated to support any identity provider and eID already supported by the broker and ensures a coherent and streamlined UX no matter what authentication options are used. 
More technically, then the Workflow Signing flow is invoked on top of a successful authentication, then using this authentication to sign the documents.
This allows flexibility and control for the integrating service, who can control reuse and age of the used authentication session, with options ranging from strict authentication pr approval to relaxed session reuse.

### Visual walkthrough
<img width="1905" height="967" alt="image" src="https://github.com/user-attachments/assets/a7a1cd9e-f295-418a-adcf-cda71785cb9a" />
<img width="1897" height="961" alt="image" src="https://github.com/user-attachments/assets/b4ddfb01-e908-4416-a965-ddd5825fbddb" />
<img width="1422" height="876" alt="image" src="https://github.com/user-attachments/assets/d6fca83c-02d5-486e-a47e-ca7377fffec3" />

## Workflow API steps and options
This section walks through the integration steps and options available. See the [provided example](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/enterprise/workflow-signing-example.html) for a more hands-on technical example.
This section is kept intentionally minimal, we refer to the Swagger documentation for more detailed API information.

### Steps

1. Create Workflow
2. When signing, get SigntextID, use as parameter for OIDC flow (signtext_id=[SigntextID]).
3. Determine if flow is completed. See resulting flow info (transaction_token) and/or use Workflow API GET API.
4. Retrieve the resulting PAdES and/or Workflow token.

### Create workflow
Create workflow:
```
POST /api/workflows
```

| Parameter |Description|
|--------| --------|
| **title**       | Workflow title |
| **pdfList:title**      | Title of PDF |
| **pdfList:pdfBase64**       | Base64 encoded bytes of PDF |
| **expiresAt**       | Expiration of workflow |
| **showStartOverviewPage**       | Controls whether the overview page is shown as a first step for end-users in the flow |

### Get workflow
Get all workflows: 
```
GET /api/workflows
```

Get single workflow:
```
GET /api/workflows/{workflowId}
```
### Sign
When starting a new signing flow / adding a signature, first retrieve a SigntextID from the Workflow API:
```
POST /api/workflows/signtextid
```
| Parameter |Description|
|--------| --------|
| **workflowId**       | Workflow ID. |
| **signerName**      | Optional name for signer. Only used if eID scheme is not providing name. |
| **signerTitle**       | Optional title for signer. Will be shown in resulting PAdES. |
| **requiredClaims**       | Optional list of required claims as <string, string> key-value pairs. Example ("idp", "mitid"). |

### Get PAdES result
When one or more signers have been added to a workflow, a PAdES can be built that seals the workflow and signers. 
Signaturgruppen uses an enterprise HSM-based signing service that seals the PAdES using an EU-trustlist certificate. The resulting PAdES is enabled for Long-term storage.
As the PAdES document can be used for both system storage and for hand-out to end-users, the API supports options for controlling the level of claims included in the PAdES taking data-privacy into account. 

```
POST /api/workflows/{workflowId}/pades
```
| Parameter |Description|
|--------| --------|
| **workflowId**       | Workflow ID. |
| **workflowTokenClaimParameters**      | Generalized parameter structure for the PAdES build API |
| **workflowTokenClaimParameters:claimOutputFilter**       | [ All, IncludeAgeClaims, Restricted (default) ]. Controls whether all, minimal or a minimal+age related claims-list of user-claims is included in the resulting PAdES. |

### Get Workflow token result
When one or more signers have been added to a workflow, a Workflow token can be built that seals the workflow and signers.
The Workflow token is signed by a dedicated HSM signing certificate. An OCSP reponse validating the validity of the signing certificate is also returned.
All available user-claims is included in the Workflow token.
```
GET /api/workflows/{workflowId}/workflowtoken
```

## PAdES and Workflow token
The two options for the final result both encapsulates the workflow and signatures added. In both variants the cryptographic digest of the original PDF documents is signed along with required claims.

## What to store for long-term verification

## Example PAdES
An example of a resulting PAdES can be [found here](https://github.com/Signaturgruppen-A-S/signaturgruppen-broker-documentation/blob/31003f0ba85f002192aac0fb2c08cbd6b76f614d/enterprise/files/2f5641e6-ea50-8558-894c-019afd1abb07.pdf).
Click the download button, to download and inspect the real PDF file. The signature of the PDF is not accepted by Adobe in this test-version, but you can inspect the attachments and see the inline content of the PAdES.
