---
title: Workflow Signing Example
layout: home
parent: Workflow Signing
grand_parent: Enterprise features
nav_order: 1
---

# Example workflow

## Create workflow
A workflow is creating by uploading 
```
POST /api/workflows/{cvr}
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
## Initiate signature
In this scenario we have an end-user and you want this end-user to go through the workflow created in the last step.
For this, the backend must retrieve a **signtext ID**, which is then used as OIDC parameter for the desired flow through Signaturgruppen Broker

Retrieve **signtext ID**: 
```
POST /api/workflows/{cvr}/signtextid
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
GET /api/workflows/{cvr}/{workflowId}
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
```
POST /api/workflows/{cvr}/{workflowId}/pades
{
  "workflowId": "[Workflow ID]",
  "workflowTokenClaimParameters": {
    "claimOutputFilter": "All"
  }
}
```
The resulting PAdES:
```
{
  "padesDocument": {
    "padesB64": "[pades bytes as Base64 string]"
  },
  "workflowResultInfo": {
    "ocspReponseIncluded": true,
    "documentCount": 2,
    "signatureCount": 2
  }
}
Retrieving the workflow token:
```
GET /api/workflows/{cvr}/{workflowId}/workflowtoken:
```

The resulting workflow token:
```
{
  "workflowToken": "[Workflow JWT token]",
  "ocspResponse": "[OCSP response for worklfow token signing cert]",
  "publicSigningCertPem": "[Signing cert for workflow token]",
  "workflowResultInfo": {
    "ocspReponseIncluded": true,
    "documentCount": 2,
    "signatureCount": 2
  }
}
```
