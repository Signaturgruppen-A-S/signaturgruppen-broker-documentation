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

| Subject | CN = SIGNATURGRUPPEN A/S - NEB Transact PP SERIALNUMBER = CVR:29915938-UID:59911227 O = SIGNATURGRUPPEN A/S // CVR:29915938 C = DK |
| --- | --- |
| Subject Thumbprint (KID) | 20595A4BE9F566771792BC3DBC7DF78FF9C36575 |
| CA Subject | CN = TRUST2408 Systemtest XXXIV CA O = TRUST2408 C = DK |
| CA Thumbprint | eeaf09230cd54e31a22872bd83cd189095921ad7 |

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

| Subject | CN = SIGNATURGRUPPEN A/S - eID Broker Signing SERIALNUMBER = CVR:29915938-UID:14521394 O = SIGNATURGRUPPEN A/S // CVR:29915938 C = DK |
| --- | --- |
| Subject Thumbprint (KID) | 8CB7F2CBABA3A57979DF96BC81DC0EAF44F30F9B |
| CA Subject | CN = TRUST2408 OCES CA IV O = TRUST2408 C = DK |
| CA Thumbprint | 5084ef33f0d4a39776281ccfdf0a9b06eea7fb9a |
