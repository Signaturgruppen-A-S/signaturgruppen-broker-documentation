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

## API Client
In order to communcate with the Signaturgruppen Broker API, you must first obtain an API client.

See the general documentation for [API clients here](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/api-integration.html).

Requesting access to the Signaturgruppen Broker general APIs set the scope **neb_api**. A request could look like

```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=9...1' \
--data-urlencode 'client_secret=p......' \
--data-urlencode 'scope=neb_api'
```

### Open Test client (DK29915938)
You can test with our open free to use Controlled Transfer Test API Client specified here. This is setup under Signaturgruppen CVR in PP. In order to test with your own CVR, create one in the PP Admin UI interface (API Client) or reach out to our support.
Note that this API client is setup under Signaturgruppen CVR **DK29915938**.

Here is a test example of a client credentials request, using this open client
```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'client_id=0dab25ae-2b9b-47bb-b8b5-353e49942780' \
--data-urlencode 'client_secret=S6ZLa9wLb7e1ZI5jNGRkvdiLJKc1F42Ng6WP6xo0ytwzEElsuuRetYIQtNOf6YWJp4JUMUTpfZJBqFfwBRdvAA==' \
--data-urlencode 'scope=neb_api'
```

This will result in a reponse on the form: 
```
{
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJhdCtqd3QifQ.eyJpc3MiOiJodHRwczovL3BwLm5ldHNlaWRicm9rZXIuZGsvb3AiLCJuYmYiOjE3NDg0MjMzNzMsImlhdCI6MTc0ODQyMzM3MywiZXhwIjoxNzQ4NDI2OTczLCJzY29wZSI6WyJuZWJfYXBpIl0sImNsaWVudF9pZCI6IjBkYWIyNWFlLTJiOWItNDdiYi1iOGI1LTM1M2U0OTk0Mjc4MCIsImNsaWVudF9vcmdfaWQiOiI4NGUxMWExNi1kOTNiLTQ4YTYtOGRhZC1iZjVlZjI2ZjMxYmUiLCJjbGllbnRfc2VydmljZV9wcm92aWRlcl9pZCI6IjljMzFlNTE4LTAyYmUtNDA4Mi04YmE2LWU5NzhjOWViNDA0OCIsImNsaWVudF9zZXJ2aWNlX2lkIjoiMjJhMDQ4NmQtOGQwOS00MWI1LWIxZjMtODYyYzMwNzQxNzY4IiwiY2xpZW50X29yZ190YXhhdGlvbl9udW1iZXIiOiJESzI5OTE1OTM4IiwiY2xpZW50X2F0X3R5cGUiOiJzZXJ2aWNlX3Rva2VuIiwianRpIjoiRDQzRjdBN0E5NERGRkMwMDQwQjk1OThGOTUxQTZDREUifQ.CWcb8ELUd5t7D11u2eIWMwDNaaZUG-rBVsxTEdmiXzfBEtzO3l6zH3FeJ1sJJlqIrs-YK_tBIJLhw0Y9g7uvLu5QLgfj0fAdGzH8nBsQDQ53svavRK1eFgQpMWJeqkPQqFTxq1m8rtZyJCynqCxeMiY_cGaAoAFdkBL2oIHCMPFnhWF1nX5WX616-C_jXi4ASj7UwQ7TiM0LgsWFGaROOLz3x4XcvCMDcYYJ76lzNgP6qiaQbGjDPKdLFAZrF5NgpmrLQOSucW11xKx908kPHdrmRkmuppK2O0AaNhgXR5sVwi-ZChOtAxXoVl5X461XZVwE9et18MnzFgXuByARfg",
    "expires_in": 3600,
    "token_type": "Bearer",
    "scope": "neb_api"
}
```

Which can then be used as **Bearer ey...461XZVwE9et18MnzFgXuByARfg** Authorization Header in the two Controlled Transfer API requests documented here. 

Note, that in a real scenario sender and receiver API clients will not be the same, and will be under different CVR contexts. If you are not the correct receiver of a CTT token, you will get a **not found** response.

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

