---
title: Ciba integration
layout: home
parent: MitID Flex app
grand_parent: Enterprise features
nav_order: 2
---

## Ciba integration
Signaturgruppen Broker has implemented the MitID Flexibility app (addon) using the Client Initiated Backchannel Authentication specification [CIBA](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#ciba). 

This section describes the technical details needed in order to integrate to the setup.

## Getting started
To familiarize yourself with the setup, it is recommended to start out using our open and free demo setup for the PP environment, which includes

* **A preconfigured demo service provider**: Approved for Flex app and approved for no channel binding.
* **Demo client credentials**: Preconfigured and openly available **client_id** and **client_secret**.

```js
service provider name: "Signaturgruppen flex app demo"
client_id: "62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9"
client_secret: "mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ=="
```

## Steps of the flow (API)

### Initiating

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid' \
--data-urlencode 'client_id=62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9' \
--data-urlencode 'client_secret=mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ==' \
--data-urlencode 'login_hint_token={"idp":"mitid","idp_params":"{"uuid":"8a9856e0-f12d-4217-b320-c8a076be9320", "referenceTextBody":"Testing MitID Flex app 😉", "ip":"1.1.1.1"}"}'
```

The login_hint_token can also be Base64 encoded:
```
--data-urlencode 'login_hint_token=eyJpZHAiOiJtaXRpZCIsImlkcF9wYXJhbXMiOiJ7InV1aWQiOiI4YTk4NTZlMC1mMTJkLTQyMTctYjMyMC1jOGEwNzZiZTkzMjAiLCAicmVmZXJlbmNlVGV4dEJvZHkiOiJUZXN0aW5nIE1pdElEIEZsZXggYXBwIPCfmIkiLCAiaXAiOiIxLjEuMS4xIn0ifQ=='
```

This would result in a response on the following form

```
{
    "auth_req_id": "0A2179A15B686CB..3AC29A5A6A-1",
    "expires_in": 300,
    "interval": 5
}
```

### Poll
Using the **auth_req_id** received in the initiation response, you can then poll for status. 

> Note that the **interval** in the init response specifies the required interval between polls in seconds.

```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid' \
--data-urlencode 'client_id=62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9' \
--data-urlencode 'client_secret=mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ==' \
--data-urlencode 'auth_req_id=0A2179A15B686CB..3AC29A5A6A-1'
```

If the flow is still pending you will receive the following response:

```
{
    "error": "authorization_pending"
}
```

When the flow has completed successfully (example):

```
{
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3BwLm5ldHNlaWRicm9rZXIuZGsvb3AiLCJuYmYiOjE3MTUwODc0NTMsImlhdCI6MTcxNTA4NzQ1MywiZXhwIjoxNzE1MDg3NzUzLCJhdWQiOiI2MmI4Y2MwMy1hYTczLTRkNmEtOTIyYy03YzhmYjBhNGExZDkiLCJhdF9oYXNoIjoiaW9NTjhaVndrdWs5czdxNk1sc1NGQSIsInN1YiI6IjdjOTQ4NGY1LTQ0YmYtNGUyOS1hNjgzLTg0NTQ4OTUxZDNiOCIsImF1dGhfdGltZSI6MTcxNTA4NzQ1MywiaWRwIjoibWl0aWQiLCJtaXRpZC51dWlkIjoiZWVlNGIyNzItNmEzMy00MGRhLWFiMGItNWNlMmFiZDQyZTc4IiwibWl0aWQuYWdlIjoiODgiLCJtaXRpZC5kYXRlX29mX2JpcnRoIjoiMTkzNi0wNC0wOCIsIm1pdGlkLmhhc19jcHIiOiJ0cnVlIiwibWl0aWQuaWRlbnRpdHlfbmFtZSI6IklsZXlhIEplbnNlbiIsIm1pdGlkLnBzZDIiOiJmYWxzZSIsIm1pdGlkLnJlZmVyZW5jZV90ZXh0Ijoib3N0ZW1hd2QgOj0pIiwibWl0aWQudHJhbnNhY3Rpb25faWQiOiI5NTgxYmFiNi05ZjhhLTQ1MjYtOWRkYy1kZmQzOTgxMjYzYjkiLCJpZHBfdHJhbnNhY3Rpb25faWQiOiI5NTgxYmFiNi05ZjhhLTQ1MjYtOWRkYy1kZmQzOTgxMjYzYjkiLCJpZHRva2VuX3R5cGUiOiJjaWJhIn0.FINs1cD2vMnnifJ0ccHi7GgMD1m4u8uZAqbVnGLncYbx3FUuW7Hl64VLQj8IAABXv8jGlX1BoYh1dKhNaJTmka-VKhR_rh1hAFrEInFvS4DYX0HKk5mVm1VOMdGcl2dzKfAjEVDq1tGcCr-HPWA7DR3Mt4vbwWJ5-ZOEJ0mA-49YZsCwNB855dcNf3PdmI0mCknUPmHLf4-ZSIfda4_tW2AmHAmJWtfWUPEQDGMLbEqkbggkETy50J06y9NM4GbczT035PW9EzeJjrqjzvgVtSy3eizRbl2W0IR95hDV1UoJTqW9xn8pt6Qs897TKbjXAmjOPphJ_jE1uGozId1RiQ",
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJhdCtqd3QifQ.eyJpc3MiOiJodHRwczovL3BwLm5ldHNlaWRicm9rZXIuZGsvb3AiLCJuYmYiOjE3MTUwODc0NTMsImlhdCI6MTcxNTA4NzQ1MywiZXhwIjoxNzE1MDkxMDUzLCJzY29wZSI6WyJvcGVuaWQiLCJtaXRpZCJdLCJjbGllbnRfaWQiOiI2MmI4Y2MwMy1hYTczLTRkNmEtOTIyYy03YzhmYjBhNGExZDkiLCJzdWIiOiI3Yzk0ODRmNS00NGJmLTRlMjktYTY4My04NDU0ODk1MWQzYjgiLCJhdXRoX3RpbWUiOjE3MTUwODc0NTMsImlkcCI6Im1pdGlkIiwiaWRwX3RyYW5zYWN0aW9uX2lkIjoiOTU4MWJhYjYtOWY4YS00NTI2LTlkZGMtZGZkMzk4MTI2M2I5IiwianRpIjoiNUIwODQ1QkREQUNGQkJERDgxMzFBNkJEOENEQkI1NDcifQ.qjU0PUBmTR90OLhm6CI6VitHr9MRVPicl2ukXBAFu5ZVI0zFTYtHKcU2yYcP5rosDEojffWdo4HxRrqx0UztGQT6RWaUBetMDv3VU0kodVJwkmmchgKL6_P7CMGnC0e2ktC-8inAiNRsPGT_P2j8p3g8cALXeSKjZKadF4e-NNpqrb37Od9cIR2otHv44IXKuUu2wxDfdeCKGYMS4pFbfB_n9aav8P1JADazX2TAkmE02N8M-J2XVFsBzzZ2G5_Us93KEYqIa2rRKGYaEWtQtYHvcfkNKor9LzB6DbIL0N3R4-IJipa2On6W9aJF6IbqnARe4PAC-iNeEtjZsx9k9A",
    "expires_in": 3600,
    "token_type": "Bearer",
    "scope": "openid mitid"
}
```

### Cancel
To cancel a flow, invoke the cancel API endpoint using the received **auth_req_id** received when initiating the flow.

```
curl --location --request DELETE 'https://pp.netseidbroker.dk/op/api/v1/ciba/0A2179A15B686CB..3AC29A5A6A-1' \
```

## Postman example

The test example is also available as a <a href="enterprise/files/CiBA PP.postman_collection.json">Postman collection here</a>.

