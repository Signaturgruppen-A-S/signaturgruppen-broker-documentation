---
title: Ciba integration
layout: home
nav_order: 1
parent: MitID Flex app
---

# Ciba integration
Signaturgruppen Broker has implemented the MitID Flexibility app (addon) using the Client Initiated Backchannel Authentication specification [CIBA](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#ciba). 

This section describes the technical details needed in order to integrate to the setup.

# Getting started
To familiarize yourself with the setup, it is recommended to start out using our open and free demo setup for the PP environment, which includes

* **A preconfigured demo service provider**: Approved for Flex app and approved for no channel binding.
* **Demo client credentials**: Preconfigured and openly available **client_id** and **client_secret**.

# Starting the flow

```url
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Cookie: X-Correlation-Id=75fc0535-ea72-44bd-9690-9cd55070b274' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid' \
--data-urlencode 'client_id=62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9' \
--data-urlencode 'client_secret=mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ==' \
--data-urlencode 'login_hint_token={"idp":"mitid","idp_params":"eyJjcHIiOiIwODA0MzYwNDY4IiwgIlJlZmVyZW5jZVRleHRCb2R5Ijoib3N0ZW1hd2QgOj0pIiwgImlwIjoiMS4xLjEuMSJ9"}'
```
