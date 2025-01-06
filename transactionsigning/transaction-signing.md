---
title: Transaction signing
layout: home
nav_order: 22
has_children: true
---

# Transaction signing
> If interested and for more information on pricing, contact sales: <salg@signaturgruppen.dk>.

## Signtext API swagger reference

| **Swagger endpoint URL** | **Description** |
| --- | --- |
| **\[Authority URL\]/ transactionsigning/api/swagger/index.html** | The swagger description of the available Signaturgruppen Broker Transaction Signing API |

## Authorization towards Signtext API

To get started using the Signtext API, your organization must first setup an API client to be able to authorize towards the Signaturgruppen Broker Signtext API.

First you need to create and setup an API client for your integration. This is done through the admin interface by setting up a new client under the correct service and service provider and by selecting “API Client” when creating the client. Setup a client secret to be used. If you do not have access to the admin interface, contact your administrator or contact Signaturgruppen to help you create and setup the client.
An existing API client can be used.

Next, request a valid service token through the Token endpoint using the following request pattern, setting the scope to “signtext_api” to retrieve a service token usable for this purpose.

Example Token endpoint request (API client)
```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'scope=signtext_api' \
--data-urlencode 'client_id=<CLIENT_ID>' \
--data-urlencode 'client_secret=<CLIENT_SECRET>'
```

Example response:

```
{
  "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJhdCtqd3QifQ…..-Sk7WRoU_BdErsN5NDfM_ FN6ahMLCDPA",
  "expires_in": 3600,  
  "token_type": "Bearer",
  "scope": "signtext_api"
}
```

Setup the service token (**access_token** in the response) as a standard authorization bearer header for the calls to the Signtext API.

> It is **highly recommended** that the service token is then cached for the duration of the lifetime of the token to reuse this for consecutive API calls with this as bearer token.

# Transaction Signing using the Signtext API

To initiate a transaction signing flow through the Signaturgruppen Broker platform, the Signtext API must be utilized.

Before starting an OpenID Connect authentication flow through the Signaturgruppen Broker platform, the signtext is uploaded to the Signtext API to generate a signtext reference, which is specified in the appropriate parameter for the OIDC flow.

First prepare the document to be signed and convert the raw bytes to BASE64. Prepare a JSON payload in the form:

```
curl --location 'https://pp.netseidbroker.dk/transactionsigning/api/v2/signtext' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJS...eyAPD6te6ohOtYCkoPpgHP2PJLfGz_oCK9w' \
--data '{
    "SignTextType": "pdf",
    "SignTextBase64": "JVBERi0xLjYNJeLjz....VPRg0K"
}'
```

The response is on the form

```
{
  "signTextId": "<SIGNTEXT_ID>"
}
```

## Start OpenID Connect flow with Signaturgruppen Broker

Now you are ready to start the signing process with the user by setting the received signtext ID in the OIDC **sigtext_id** parameter, like in the following example:

```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=0a775a87-878c-4b83-abe3-ee29c720c3e7&redirect_uri=https://openidconnect.net/callback&idp_values=mitid&prompt=login&scope=openid+mitid+transaction_token&response_type=code&signtext_id=pdf_ab79c129-099e-4800-826b-8c63891e68f3&prompt=login
```


Note that in the MitID flow you can control the “ACTION” shown in MitID by using the idp_param called action_text. Eg. In order to indicate a signing flow the action_text, “SIGN”, could be utilized:

```
https://pp.netseidbroker.dk/op/connect/authorize
?client_id=<CLIENT_ID>
&prompt=login
&redirect_uri=<REDIRECT_URI>
&scope=openid%20mitid%20ssn
&response_type=code
&state=75..43dad4b
&idp_values=mitid
&idp_params=%7B%22mitid%22%3A%7B%22action_text%22%3A%22SIGN%22%7D%7D
&signtext_id=<SIGNTEXT_ID>
&prompt=login
```


### Usage of Prompt=login

Per default, Signaturgruppen Broker will keep a session with the end-user which will be reused for authentication requests if able based on the authentication request parameters.

The transaction text flow is a sub-flow available to select authentications and is considered an optional parameter for these flows which is activated for these flows only when the user is doing an interaction authentication with a supported identity provider like MitID.

So, specifying the transaction signing **signtext_id** parameter, will not in itself force the signing flow if the end-user already has an active session which for the requested authentication can be automatically completed. The transaction token will reflect what the end-user completed and saw, and thus in an automatically completed authentication without any user interaction, there will not be a transaction signing SHA256 or any transaction actions claims returned.

To force the authentication for the end-user, the **prompt=login** authentication parameter can be specified, which will ensure that the end-user is always presented for the requested authentication and transaction signing flow.

# Transaction Signature Verification

After the user completes the MitID transaction the resulting transaction token will be available as jwt from the token endpoint.

To secure the maximal strength of the transaction perform the following verifications

1. Verify token signature towards signing key (in JWKS)
2. Verify certificate in JWKS (at least when it changes)
3. Check enclosed OCSP response for updated jwt signing certificate status
4. Verify contents of transaction token
    1. Expected signer
    2. Audience matches service provider
    3. Timestamp is within expected time
    4. transaction_text_sha256 matches sha256 fingerprint of document.
5. Store original document, transaction token, certificate and OCSP for later verification.

If utilizing the PAdES wrapping service extension for transaction signing flows, then all previous verifications will be optional, as the resulting PAdES will contain all relevant files as attachment and will be sealed by the Signaturgruppen PAdES sealing service and certificate. Instead, the received PAdES should be stored and can be transferred to the active signers of the document as well. See the section for the PadES wrapping API for details.

# Transaction Token Contents Example

The received transaction tokens resulting from the signing process can be decoded as normal JWT. Below is a decoded example.

The header below indicates the Key Identifier (kid) used to sign the token. Note that the kid can be used to localize the key in the broker jwks-endpoint (<https://pp.netseidbroker.dk/op/.well-known/openid-configuration/jwks>)

```
{  
  "alg": "RS256",  
  "kid": "20595A4BE9F566771792BC3DBC7DF78FF9C36575",  
  "typ": "JWT"  
}
```

The body of the token holds the information below (example)

```
{  
  "mitid.uuid": "6867f..2118",  
  "mitid.age": "17",  
  "mitid.date_of_birth": "2005-06-03",  
  "mitid.identity_name": "Edvard Nielsen",  
  "mitid.transaction_id": "bfa627f...cddf99c",  
  "amr": "code_app",  
  "loa": <https://data.gov.dk/concept/core/nsis/Substantial>,  
  "acr": <https://data.gov.dk/concept/core/nsis/Substantial>,  
  "ial": <https://data.gov.dk/concept/core/nsis/Substantial>,  
  "aal": <https://data.gov.dk/concept/core/nsis/Substantial>,  
  "identity_type": "private",  
  "idp": "mitid",  
  "auth_time": "1663143384",  
  "sub": "5cb6c9...2bf7b",  
  "transaction_id": "0ee595...9901ad4a",  
  "redirect_uri": <https://brokerdemo-pp.signaturgruppen.dk/signin-oidc>,  
  "mitid.psd2": "false",  
  "transaction_text_type": "pdf",  
  "transaction_text_sha256": "BfjQQwVirWIjKj...g/Dak=",  
  "transaction_actions": \[  
    "mitid.login",  
    "mitid.transaction_signing"  
  \],  
  "signing_cert_ocsp_nonce": "LR19s2V2kd25....e58WRUOBerYqg1CZRCfBUbA97EJd0tY=",  
  "nbf": 1663143385,  
  "exp": 1852532185,  
  "iat": 1663143385,  
  "iss": <https://pp.netseidbroker.dk/op>,  
  "aud": "14148e23-1d32-4d5b-a7ef-8935fdf65836"  
}
```

# Transaction signing PAdES wrapping API

> _Only usable for PDF transaction signing flows._

Service providers might have the requirement that a PDF receipt can be generated and given to the user for later reference. Unfortunately, most common PDF viewers today do not have signature capabilities and hence embedded signatures might not be visible. Hence, we support the option for transforming a transaction token into a user-view-friendly PDF. The addition is based on a new endpoint that can be called with the received transaction token with transaction signing of an PDF to create a sealed PAdES (signed PDF) wrapping up all required documents and with an additional signer page added. This PAdES will be self-contained and can be handed out to relevant end-users and systems for verification and storage of the transaction signing flow and will be self-contained for usage in any legal dispute.

After a PDF transaction signing is completed the service provider uses the received transasction token at the PAdES wrapping API endpoint, which also requires authorization as previously described.

Example PAdES wrap request
```
curl --location 'https://pp.netseidbroker.dk/transactionsigning/api/v1/pades' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer <SERVICE_TOKEN>' \
--data '{    
    "JwtTransactionTokensSerialized": 
        "[INSERT TRANSACTION TOKEN JWT HERE]"
}'
```
The response is on the form
```
{
  "padesB64": "string"
}
```

# Transaction signing examples

## Sample flow with Text

1) Prepare text to Base64:

```
text: Simple document to be signed
base64: U2ltcGxlIGRvY3VtZW50IHRvIGJlIHNpZ25lZA==
```

2) Upload of text

```
curl --location 'https://pp.netseidbroker.dk/transactionsigning/api/v2/signtext' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJS...eyAPD6te6ohOtYCkoPpgHP2PJLfGz_oCK9w' \
--data '{
    "SignTextType": "text",
    "SignTextBase64": "U2ltcGxlIGRvY3VtZW50IHRvIGJlIHNpZ25lZA=="
}'
```

3) Receive id

```
{  
  "signTextId": "text_e2619d37d45cbf0d78f2f63d5eb..cc53d8e09b0c8"  
}
```

4) Start flow with request:

```
https://pp.netseidbroker.dk/op/connect/authorize
?client_id=<CLIENT_ID>
&redirect_uri=https://openidconnect.net/callback
&idp_values=mitid
&prompt=login
&scope=openid+mitid+transaction_token
&response_type=code
&signtext_id=text_e2619d37d45cbf0d78f2f63d5eb..cc53d8e09b0c8
```
