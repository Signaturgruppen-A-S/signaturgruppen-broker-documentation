---
title: Environments
layout: home
nav_order: 7
---

## Environments

This section defines the available environments available from NEB and their respective URLs and certificates.
Security related information like TLS and VOCES3 signing certificate DN and CA root information, will also be available for each environment.

### Preproduction environment – https://pp.netseidbroker.dk/op

| Variable | Values |
| --- | --- |
| Authority URL | https://pp.netseidbroker.dk/op |
| Discovery endpoint | https://pp.netseidbroker.dk/op/.well-known/openid-configuration |
| Signaturgruppen Broker MitID Broker ID | f81b4f9a-2ca2-49ec-ba52-654de7edfcdc |

#### Token signing certificate 

| Subject | CN = Signaturgruppen Broker Token Signing 1 PP Env C = DK |
| --- | --- |
| Subject Thumbprint (KID) | 048058BB59F4D3007045896FD488CE81F4EB4923 |
| CA Subject | CN = Signaturgruppen Broker Token Signing Root PP Env C = DK |
| CA Thumbprint | 1beb2d3df149237427ae40abe524882a7ebb2ddb |

#### SSL certificate

| Subject | CN = pp.netseidbroker.dk |
| --- | --- |
| CA Subject | CN = R3 O = Let's Encrypt C = US |
| CA Thumbprint | a053375bfe84e8b748782c7cee15827a6af5a405 |

#### Transaction token signing certificate

| Subject | CN = NEB Transact PP, SERIALNUMBER = UI:DK-O:G:d09df20d-26f0-4ba0-bdd1-2cb4f9524683, O = SIGNATURGRUPPEN A/S, 2.5.4.97 = NTRDK-29915938, C = DK |
| --- |-------------------------------------------------------------------------------------------------------------------------------------------------|
| Subject Thumbprint (KID) | D8A5D9D7EDE992F1A0F8E499E992E9DA805A05D8                                                                                                        |
| CA Subject | CN = Den Danske Stat OCES udstedende-CA 1, OU = Test - cti, O = Den Danske Stat, C = DK                                                         |
| CA Thumbprint | 72347BAE1745688942D49DFF5F2C80538B5B023B                                                                                                        |

### Production environment – https://netseidbroker.dk/op

| Variable | Values |
| --- | --- |
| Authority URL | https://netseidbroker.dk/op |
| Discovery endpoint | https://netseidbroker.dk/op/.well-known/openid-configuration |
| Signaturgruppen Broker MitID Broker ID | a9df260d-42c6-4e4c-85a5-681423673a78 |

#### Token signing certificate 

| Subject | CN = Signaturgruppen Broker Token Signing 1 Prod Env C = DK |
| --- | --- |
| Subject Thumbprint (KID) | 353E2FE9191CDEC22C8B52D2B7A82A2DAA50642E |
| CA Subject | CN = Signaturgruppen Broker Token Signing Root Prod Env C = DK |
| CA Thumbprint | fa516c6bb2d07103a54fe4cd6ded4aed30b360f7 |

#### SSL certificate

| Subject | CN = netseidbroker.dk |
| --- | --- |
| CA Subject | CN = R3 O = Let's Encrypt C = US |
| CA Thumbprint | a053375bfe84e8b748782c7cee15827a6af5a405 |

#### Transaction token signing certificate

| Subject | CN = eID Broker Signing, SERIALNUMBER = UI:DK-O:G:eca57a69-98a6-4b9a-9293-bc4be1c3aa06, O = SIGNATURGRUPPEN A/S, 2.5.4.97 = NTRDK-29915938, C = DK |
| --- |------------------------------------------------------------------------------------------------------------------------------------------------|
| Subject Thumbprint (KID) | B820F52255649CE8E039F4E1B9F96F7C1E108C08                                                                                                       |
| CA Subject | CN = Den Danske Stat OCES udstedende-CA 1, O = Den Danske Stat, C = DK                                                         |
| CA Thumbprint | 0B7F84237B7423B800495761BBC8D34F1AB83B50                                                                                                       |
