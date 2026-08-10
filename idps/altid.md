---
title: AltID
layout: home
parent: Identity providers
has_children: false
nav_order: 20
---

# AltID
AltID is a Danish national wallet implementation - https://digst.dk/it-loesninger/altid/.
It is currently not a fully verified EU Digital Identity Wallet, however the interface and design remains similar to an EU Digital Identity Wallet. The goal is also for AltID to eventually become a EU Digital Identity Wallet.

AltID is however a verified age verification service as can be seen by the official [EU Age Verification trust list](https://eidas.ec.europa.eu/efda/trust-services/browse/av-tl).

We distinguish between two primary flows: Age Verification and Person Identification Data (PID). The reason for this distinction is that the Age Verification flow does not require a Relying Party Registration. This makes it easy to use and very accessible.

## Age Verification
This flow supports any Proof of Age Attestation Services on the official [EU Age Verification trust list](https://eidas.ec.europa.eu/efda/trust-services/browse/av-tl) and is not just limited to AltID.

The age verification flow is a data minimal flow where the user is simply asked whether they are above a certain age.
This simple flow is easy to set up and requires minimal configuration.
To get started you need to enable the EU wallet in the broker administration site. 
Once enabled, you select the service provider you wish to use for your service and then configure the EU Digital Identity Wallet identity provider.
Ensure it is enabled and select unsigned as the type for the service provider IdP configuration. Any client under this service provider now supports the flow.

To request age verification, include one or more `age_over:{age}` values in the `scope` parameter.

For example:

```text
scope=openid age_over:18 age_over:21
```

If the wallet can satisfy the request, it will return a corresponding claim for each requested scope:

```json
{
  "age_over_18": true,
  "age_over_21": true
}
```

> Different identity providers may support different age thresholds. Requests for unsupported ages may be ignored and not returned.

### Supported Age Verification scope values

| Scope | Claim            | Description |
|----------------|------------------|-------------|
| `age_over:{age}` | `age_over_{age}` | Requests verification that the holder is over the specified age. Multiple `age_over:{age}` scopes may be included in a single request. A corresponding claim is returned for each requested scope that is successfully verified. |

### AltID supported ages

AltID currently supports the following age thresholds:

| Provider | Supported age_over scopes                                              |
|----------|------------------------------------------------------------------------|
| **AltID** | **13**, **15**, **16**, **18**, **21**, **23**, **25**, **27**, **67** |

For more information on the Age Verification flow and other supported identity providers see [Age Verification](../ageverification/age_verification_eu.md).

## Person Identification Data (PID)
Requesting person identification data (PID) requires more configuration. This requires a Relying Party certificate and verifier info from a Relying Party Registry.
For AltID this certificate should be a system certificate issued from MitID erhverv and added to AltIDs Relying Party Registry. Details of how this can be done can be found at [AltID modtagerregister](https://modtager.tegnebog.dk/).
Once added to the AltID Relying Party Registry it must be added to the broker administration site along the verifier info provided by the AltID Relying Party Registry. 

In the broker administration site this requires configuring the certificate as well as the verifier info. 
This is done by selecting the EU Digital Identity Wallet configuration on the specific service provider. Selecting the type to be Relying Party Registration and 
then uploading the certificate and verifier info.

The verifier name provided is what will be visible to the user when they are asked to provide their Person Identification Data (PID).
The name must be present in the JWT for it to be valid. 


### AltID Supported PID scopes

Each requested scope returns a claim with the same name in the credential.

| Scope / Claim | Description |
|---------------|-------------|
| `full_name` | Full name of the user. |
| `family_name` | Last name(s) of the user. |
| `given_name` | First and middle name(s) of the user. |
| `birth_date` | The user's date of birth. |
| `birth_place` | The area where the user's birth was registered, such as a parish, hospital, or other location. |
| `nationality` | One or more Alpha-2 country codes representing the user's nationalities. `DK` is listed first if present. |
| `resident_address` | The user's full residential address. |
| `personal_administrative_number` | The user's CPR number. |

### Metadata scopes
| Scope / Claim | Description |
|---------------|-------------|
| `expiry_date` | Expiry date of the source document or data the credential is based on (not the credential itself). |
| `issuing_authority` | The authority that issued the PID attestation. For AltID this is always `Digitaliseringsstyrelsen`. |
| `issuing_country` | Country of the issuing authority. For AltID this is always `DK`. |

### Age Verification scopes
Age verification scopes are also supported in this flow, see [Supported Age Verification scope values](#supported-scope-values)