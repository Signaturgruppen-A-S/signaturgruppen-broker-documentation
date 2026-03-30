---
title: Age Verification
layout: home
has_children: true
nav_order: 100
---

# Age Verification
EU Age Verification (AV) with possibility for additional AV providers and flexible integration.

> For MitID only or legacy integrations, see the docs [here](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/ageverification/age_verification.html).

> Under development. Contact us for interest.

## Introduction to Age Verification
To support Age Verification (AV), Signaturgruppen Broker has implemented a streamlined and robust interface that allows for an easy, compliant and flexible integration from any system and adaptable to any workflow. 

Data minimization is primary - only the AV result is returned. 

The integration interface is minimal and stable, while the administrative UI allows for flexible customization for integrations. This supports stable integrations that is easily adaptable for changes over time, without the need to changes to the integrating service.

### The European Digital Identity Wallet (EUDI) AV and alternatives
The primary AV mechanism supported is the The European Digital Identity Wallet (EUDI) supported AV flow, which is specifically tailored to be data minimalistic, secure, anonymous and widely available cross borders across the EU. This includes support for national AV solutions under the EU AV scheme, such as the danish AltID.

Other AV providers is supported and can be configured and enabled for the integration via the administrative UI.

List of supported AV providers

| AV provider | IDP identifier | Description |
|--------|------|--------|
| EU Age Verification       | eu_av     | EUDI AV. Supports any national implementation of the EUDI AV scheme, such as the danish **AltID**.        |
| AltID       | eu_av      | AltID is the danish national wallet based on EUDI.         |
| MitID       | mitid      | MitID is the danish national ID.         |
| e-Boks       | eboks      | EUDI-style wallet age verification: reusable age proofs through e-Boks ID/e-Wallet with selective disclosure for online and physical use cases. |

## Supported AV integration variants
Signaturgruppen Broker supports three primary integration protocols for AV integration, namely OpenID Connect (OIDC), OpenID Connect Client-Initiated Backchannel Authentication Flow (CIBA) and a browser initiated iframe variant. 
These variants cater to different integration requirements and needs and provides support for virtually all possible integration scenarios. 

* The OIDC protocol support a standards driven, browser-first integration that allows for a wide range of applications to easily integrate.
* The CIBA protocol support a backend initiated workflow with flexible options and ways to receive the result directly to your backend.
* The iframe protocol supports a browser initiated integration that allow for easy and flexibile integration into existing web-based setups that have a harder time to adjust their workflows using the OIDC or CIBA protocol. The iframe variant better support workflows in which you do not want to redirect the end-user away from the current context and do not want to handle OIDC pop-up style handling.

## AV scopes
The AV variants all share the same request format for the AV request utilizing the **scope** parameter. Here, you can request one or more AV scopes, which directly maps to the requested age credentials/proofs that you request. 

All available AV providers support a set of standard AV age brackets such as 15, 16, 18 and 21. These are requested via the parameterized **av:[age]** scope value.

The av scope has a generic parameterization in our integration to support the most flexible integration. We will ensure to fail-early or adapt the request to the specific provider utilized in the request.

### Examples
Example for OIDC and CIBA:
```
scope=openid av:16 av:18
```

Example for iframe integration:
```
scope=av:16 av:18
```

