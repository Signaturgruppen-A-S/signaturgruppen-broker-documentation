---
title: MitID Controlled Transfer
layout: home
parent: Enterprise features
nav_order: 9
---

# MitID Controlled Transfer

### Overview
MitID Controlled Transfer Token (CTT) allows an authenticated session from one service provider (SP A) to be securely transferred to another service provider (SP B). This mechanism ensures authentication integrity is maintained and supports seamless session sharing.

### Process

#### Requesting a Controlled Transfer Token
- Service Provider A requests a Controlled Transfer Token (CTT) from Signaturgruppen Broker.
- Request:
  ```plaintext
  POST https://pp.netseidbroker.dk/op/api/v1/MitId/ControlledTransfer/tokenExchangeCode
  ```
- Headers:
  ```json
  {
    "Content-Type": "application/json",
    "Authorization": "Bearer {access_token}"
  }
  ```
- Body:
  ```json
  {
    "targetBroker": "{UUID of target broker}",
    "targetServiceProvider": "{UUID of target Service Provider (SP B)}",
    "transferTokenText": "some text"
  }
  ```
- The response includes a `transferTokenExchangeCode`, which must be passed to SP B.

#### Completing the Transfer at Service Provider B
- SP A redirects the user to SP B with the token.
- SP B exchanges the token at the broker endpoint:
  ```plaintext
  GET https://pp.netseidbroker.dk/op/connect/authorize?
  client_id={client_id}&redirect_uri={callback_url}&scope=openid mitid userinfo_token
  &response_type=code&state={state_value}&idp_values=mitid&idp_params={transfer_token_details}
  ```
- SP B receives an authentication response, completing the transfer.

### Flow Diagram
![](../images/mitid-CTT-flow.png)
---


## Comparison of MitID CTT and Signaturgruppen Controlled Transfer

| Feature             | MitID Controlled Transfer Token (CTT)                   | Signaturgruppen Controlled Transfer                                                                           |
|---------------------|---------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| **Purpose**         | Secure session transfer between SPs using MitID         | General session transfer with extended customization                                                          |
| **Broker Involved** | Signaturgruppen Broker                                  | Signaturgruppen Broker                                                                                        |
| **Data Transfer**   | Only authentication session (CPR cannot be transferred) | Can include user attributes (claims, etc.), but the sender must ensure appropriate consent for data transfer. |
| **Flexibility**     | Limited to MitID authentication flow                    | Can be adapted to various scenarios                                                                           |

---

## Summary
MitID Controlled Transfer Token (CTT) is a standardized method specifically for MitID session transfers, ensuring authentication integrity between service providers. On the other hand, Signaturgruppen Controlled Transfer offers greater flexibility, allowing the inclusion of additional user data and broader use cases beyond MitID authentication. Both methods ensure secure and efficient session transfers between service providers.

