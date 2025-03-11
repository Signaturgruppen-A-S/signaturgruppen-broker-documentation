---
title: Controlled Transfer
layout: home
parent: Enterprise features
nav_order: 8
---

# Signaturgruppen Controlled Transfer

### Overview
Signaturgruppen Controlled Transfer provides a flexible session transfer mechanism between service providers. Unlike MitID Controlled Transfer, which has strict data transfer limitations, Signaturgruppens Controlled Transfer allows the sender to include any relevant data, provided they have obtained the necessary consent. Unlike MitID CTT, this method allows additional customization, including transferring user attributes beyond authentication details.

### Process

#### Creating a Controlled Transfer Token
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
- The broker returns a **Controlled Transfer Token (CTT)**.

#### Exchanging the CTT for Authentication
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
- SP B receives authentication session details in a signed JWT token.

### Flow Diagram
![](../images/SG-CTT-Flow.png)
---

### Consent Requirement
In Signaturgruppens Controlled Transfer, it is the sender's responsibility to ensure that the required consent has been obtained before transferring any data. Signaturgruppens Broker does not enforce or verify consent handling.

### Security Considerations
- Session data is based on the authentication at SP A.
- SP B should verify session details and consider additional user verification for sensitive operations.
- The transfer token has a short lifespan and can only be used once.

---

