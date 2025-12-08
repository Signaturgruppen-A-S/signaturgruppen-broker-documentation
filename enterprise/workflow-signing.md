title: Workflow Signing
layout: home
parent: Enterprise features
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

### MitID test identities
We have setup the demo to utilized MitID PP test identities and setup the demo to use [Signaturgruppen Broker Simulation](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/enterprise/simulation.html) for easier and faster testing. 

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

## Technical overview
Workflow Signing consist of an API and setting one additional parameter for the Signaturgruppen Broker OpenID Connect interface; this together allow for the creation of workflows that supports PDF document signing for one or more signers, decoupled over time with various requirements and restrictions.

It starts with the creation of a workflow by uploading one or more PDF documents for signing together with relevant parameters for the specific flow like workflow title, controlling UI experience etc.

Anytime a new signer is ready to sign, a signtext ID is retrieved from the API, which is used in the OpenID Connect integration towards Signaturgruppen Broker. 

Upon successful signature, the integrating system has full insight into what happened and by whom and is able to retrieve the results in both PAdES and a signed JWT for long-term storage formats.

The integrating service has full control over signers, can add more dynamically and is able to retrieve the resulting documents and tokens at any time.

### Authentication, approval and OpenID Connect
The Workflow Signing flows works as part of the Signaturgruppen Broker platform and has been integrated to support any identity provider and eID already supported by the broker and ensures a coherent and streamlined UX no matter what authentication options are used. 
More technically, then the Workflow Signing flow is invoked on top of a successful authentication, then using this authentication to sign the documents.
This allows flexibility and control for the integrating service, who can control reuse and age of the used authentication session, with options randing from strict authentication pr approval to relaxed session reuse.

### Visual walkthrough
<img width="1905" height="967" alt="image" src="https://github.com/user-attachments/assets/a7a1cd9e-f295-418a-adcf-cda71785cb9a" />
<img width="1897" height="961" alt="image" src="https://github.com/user-attachments/assets/b4ddfb01-e908-4416-a965-ddd5825fbddb" />
<img width="1422" height="876" alt="image" src="https://github.com/user-attachments/assets/d6fca83c-02d5-486e-a47e-ca7377fffec3" />


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
Additional parameters are available at this step, including the ability to restrict the signer to specific claims. See the API documentation for details on this.

In response, your backend will receive: 
```
{
  "workflowId": "workflowid1234",
  "signtextId": "signtextid_random_1",
  "expiresAt": "[expires]"
}
```

Now, you can specify the "signtext_id=[signtextId]" OIDC parameter for any relevant flow through the broker. 

Lets now assume that you want a MitID signature and are doing the integration to Signaturgruppen Broker using the PAR OpenID Connect variant. A simple integration could look like:

```
curl --location 'https://pp.netseidbroker.dk/op/connect/par' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'redirect_uri=[redirect_uri]' \
--data-urlencode 'client_id=[client_id]' \
--data-urlencode 'client_secret=[client_secret]' \
--data-urlencode 'response_type=code' \
--data-urlencode 'scope=openid mitid transaction_token' \
--data-urlencode 'max_age=600' \
--data-urlencode 'signtext_id=signtextid_random_1' \
--data-urlencode 'idp_values=mitid'
```
Giving a response like
```
{
    "request_uri": "urn:ietf:params:oauth:request_uri:C834....7B4",
    "expires_in": 600
}
```
Then simply navigate the end-user to
```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=[client_id]&request_uri=urn:ietf:params:oauth:request_uri:C834....7B4
```
When the end-user completes his OIDC flow, the workflow has been updated and data + resulting PAdES and/or signed workflow token can be retrived. 
If you additionally specify the **transaction_token** scope for the OIDC request, you will receive a signed token for the specific OIDC flow giving a strong binding to what the user saw and did for that particluar OIDC flow. The transaction token is optional for the overall workflow, but can be used for more tight verification and audit requirements for each individual OIDC flow completed.
Here is an example of such a **transaction_token**:
```
{
  "mitid.transaction_id": "e4c6e9cd-8e69-47b2-8c7d-7b95f873d755",
  "mitid.uuid": "e...8",
  "mitid.age": "89",
  "mitid.date_of_birth": "1936-04-08",
  "mitid.has_cpr": "true",
  "mitid.identity_name": "Ileya Jensen",
  "amr": "code_app",
  "loa": "https://data.gov.dk/concept/core/nsis/Substantial",
  "ial": "https://data.gov.dk/concept/core/nsis/Substantial",
  "aal": "https://data.gov.dk/concept/core/nsis/Substantial",
  "identity_type": "private",
  "idp_identity_id": "e..8",
  "idp": "mitid",
  "acr": "https://data.gov.dk/concept/core/nsis/Substantial",
  "auth_time": "1764593359",
  "sub": "9..5",
  "transaction_id": "98150b00-1729-4f40-9c14-6940cead4bc9",
  "redirect_uri": "https://brokerdemo-pp.signaturgruppen.dk/signin-oidc",
  "transaction_actions": [
    "mitid.login",
    "workflow_finalized"
  ],
  "workflow_signature_transaction_id": "f73064fe-2d6e-4635-adca-e9623734c5ed",
  "workflow_id": "workflowid1234",
  "workflow_pdf_title_and_sha256": "{\"Agreement for Digital Signing\":\"f2888bc8aae9cbc0f691a6847ea9f0401785959b17b1a9f9a84d11e54d3614c8\",\"Demo Data \\u0026 Permissions Overview\":\"3ccc0e3a4d7dacc3dfbc6150158c2000c690964c2754d4f3f9ce5365dfd5c8fb\"}",
  "transaction_client_ip": "1..9",
  "nbf": 1764593469,
  "exp": 1953895869,
  "iat": 1764593469,
  "iss": "https://pp.netseidbroker.dk/op",
  "aud": "14148e23-1d32-4d5b-a7ef-8935fdf65836"
}
```
The service provider is able to retrieve the status of a workflow at any time:

```
GET /api/workflows/{workflowId}
```
Which will result in a response like:
```
{
    "id": "36....01",
    "created": "2025-12-03T08:19:05.4942269+00:00",
    "title": "[Title]",
    "organizationTin": "DK...",
    "expiresAt": "2025-12-04T09:18:00+00:00",
    "signatures": [
        {
            "id": "d24....b9",
            "created": "2025-12-03T08:19:22.4973269+00:00",
            "serviceProvidedSignerName": null,
            "signatureTransactionId": "5e7...40",
            "userClaims": {
                "mitid.identity_name": "Anker Andersen",
                "loa": "https://data.gov.dk/concept/core/nsis/Substantial",
                "ial": "https://data.gov.dk/concept/core/nsis/Substantial",
                "idp_identity_id": "e799.......52",
                "idp": "mitid",
                "auth_time": "1764749954"
            },
            "ipAddressOfUser": "...9"
        }
    ]
}
```
This gives a status of the workflow and a list of the signatures that have been added to the document.

As soon as there is at least one signature on a workflow, the resulting PAdES can be generated:
> NOTE: this endpoint will likely change or a new added, which allow for option parameters
```
GET /api/workflows/{workflowId}/pades
```
Getting the PAdES back:
```
{
  "padesB64": "[pades bytes as Base64 string]"
}
```
