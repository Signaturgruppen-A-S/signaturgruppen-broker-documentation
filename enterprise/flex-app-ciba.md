---
title: Ciba integration
layout: home
nav_order: 1
parent: MitID Flex app
---

# Ciba integration
Signaturgruppen Broker has implemented the MitID Flexibility app (addon) using the Client Initiated Backchannel Authentication specification [CIBA](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#ciba). 

This section describes the technical details needed in order to integrate to the setup.

```url
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: X-Correlation-Id=75fc0535-ea72-44bd-9690-9cd55070b274' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid' \
--data-urlencode 'client_id=...' \
--data-urlencode 'client_secret=...' \
--data-urlencode 'login_hint_token={"idp":"mitid","idp_params":"..."}'
```
