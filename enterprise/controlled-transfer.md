---
title: Controlled Transfer
layout: home
parent: Enterprise features
nav_order: 8
---

# Signaturgruppen Controlled Transfer

## Overview
Signaturgruppen Controlled Transfer provides a flexible session/claims transfer mechanism between service providers (SP). Signaturgruppens Controlled Transfer allows the sender to include any relevant data, provided they have obtained the necessary consent.

The sender service provider initiates the protocol by an API request towards Signaturgruppen Broker using a secure API client, specifying the receiver and payload, which will result in a Controlled Transfer Token (CTT). The CTT is a one-time time-limited token, which is then transfered to the receiver service provider. The receiver service provider can then exchange the CTT, using a secure API client, to a Signaturgruppen Broker signed ID token which contains the relevant claims and sender information.

For conceptual clarity we will describe the transfer of a session in the following documentation - but the protocol is agnostic to the concept of a session - in principle any data is transfered and the receiver does not have to setup a session.

## Flow Diagram
![](../images/SG-CTT-Flow.png)
---

## Swagger API reference
The Swagger API reference is found for the PP environment under the "Controlled Transfer" section at:

```
https://pp.netseidbroker.dk/op/swagger/index.html
```

## Process


### Creating a Controlled Transfer Token
- SP A initiates a request to Signaturgruppen Broker:
  ```plaintext
  POST https://pp.netseidbroker.dk/op/api/v1/controlledTransfer/create
  ```
- Request body:
  ```json
  {
    "targetTin": "DK000000000",
    "senderTraceIdentifier": "unique-trace-id",
    "claims": { "claimType1": "value1", "claimType2": "value2" }
  }
  ```
- The broker returns a **Controlled Transfer Token (CTT)**:
  ```
  {
    "id": "string",
    "ctt": "string",
    "senderTraceIdentifier": "string"
  }
  ```

### Exchanging the CTT for Authentication
- Signaturgruppens Broker ensures that only a secure API client from the specified target CVR can exchange the token. This guarantees that only the correct recipient can complete the transaction and verify the sender.
- SP B exchanges the received CTT for a valid authentication session:
  ```plaintext
  POST https://pp.netseidbroker.dk/op/api/v1/controlledTransfer/exchange
  ```
- Request body:
  ```json
  {
    "ctt": "{received_CTT}"
  }
  ```
- SP B receives authentication session details in a signed JWT token:
  ```
  {
    "id": "string",
    "idToken": "string",
    "senderTraceIdentifier": "string"
  }
  ```
- The **ID token** is signed by default signing key found at Signaturgruppen Broker Discovery endpoint:
  ```
  {
    "iss": "https://pp.netseidbroker.dk/op",
    "nbf": 1748422689,
    "iat": 1748422689,
    "exp": 1748422989,
    "aud": "DK29915938",
    "ct_claims": "{\"claim_1\":\"value1\",\"claim_2\":\"value2\"}",
    "ct_id": "f06ed0..6d2",
    "ct_sender": "DK..8",
    "ct_sender_traceId": "trace-id-1234"
  }
  ```
The **ID token** contains the **ct_sender** CVR which identifies the sender.

## Consent Requirement
In Signaturgruppens Controlled Transfer, it is the sender's responsibility to ensure that the required consent has been obtained before transferring any data. Signaturgruppens Broker does not enforce or verify consent handling.

## Security Considerations
When setting up Controlled Transfer some details on security should be considered. 
- Session data is based on the authentication at SP A. If regulatation mandates that a standardized security level is required a step-up-authentication like MitID can be performed.
- SP B could consider additional user authentication for sensitive operations based on a risk analysis.
- The transfer token has a short lifespan and can only be used once.
- SP B should verify session details and the JWT token signature.

---

