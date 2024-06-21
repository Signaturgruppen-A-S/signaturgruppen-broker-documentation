---
title: NemLog-in3 integrations
layout: home
parent: Transaction signing
nav_order: 25
---

# NemLog-in3 integrations

The signtext api integrates and exposes some functionality supplied by NemLog-in3.

## Cpr matches signer ID

NemLog-in3 provides a service for qualified signatures. Within the signed AdES documents, NemLog-in3 supplies an unique identifier which can be matched to a cpr using NemLog-In webservices.
We provide an intgration to this service. The integration is part of signtext api and can be accessed using the same authorization.

### CPR match signer ID API example

Invoke the match, setting both CPR and the unique identifier found in the received signed AdES document.
```
curl --location 'https://pp.netseidbroker.dk/transactionsigning/api/v2/signtext/nemlogin/cprmatch' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer eyJhbGciOiJS...eyAPD6te6ohOtYCkoPpgHP2PJLfGz_oCK9w' \
--data '{
    "cpr": "1234567890",
    "nl3Identity": "UI:DK-P:G:894715f7-4261-4443-9f1c-d45f1f21b222"
}'
```

The response is on the following form, informing if the CPR and signer ID matches:
```
{
  "match": "bool"
}
```
