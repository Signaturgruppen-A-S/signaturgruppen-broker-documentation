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

### EU AV and alternatives
The primary AV mechanism supported is the European Wallet supported AV flow, which is specifically tailored to be data minimalistic, secure, anonymous and widely available cross borders across the EU. This includes support for national AV solutions under the EU AV scheme, such as the danish AltID.

Other AV providers such as the danish MitID and e-Boks (non-exhaustive list) is supported and can be configured and enabled for the integration via the administrative UI.

## Supported AV integration variants
Signaturgruppen Broker supports two primary integration protocols for AV integration, namely CIBA and a browser initiated iframe variant. 
These two variants cater to different integration requirements and needs and provides support for most integration scenarios. 

The CIBA protocol support a backend initiated well documented workflow with flexible options and ways to receive the result directly to your backend.
The iframe protocol supports a browser initiated integration that allow for easy and flexibile integration into existing web-based setups that have a harder time to adjust their workflows using the CIBA protocol.

