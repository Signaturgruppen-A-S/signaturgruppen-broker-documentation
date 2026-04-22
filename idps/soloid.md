---
title: SoloID
layout: home
parent: Identity providers
has_children: false
nav_order: 4
---

# SoloID

SoloID is an identity provider that offers range of authenticators to ease the authentication process. It is an identity provider capable of providing an NSIS substantial level of authentication comparable to the most common MitID level of authentication.

The flow can be tested [here](https://brokerdemo-pp.signaturgruppen.dk/Home/LoggedInSuccess?mitidEnabled=false&mitidErhvervEnabled=false&nemloginEnabled=false&lokalIdpEnabled=false&passkeyidEnabled=false&soloidpEnabled=true&eboksEnabled=false&mobilepayEnabled=false&euidEnabled=false&altidEnabled=false&soloIdpAuthenticationMethods=mitid%2Cpasskey&prompt=login&max_age=999999&simulation=999999&language=da-DK&scope=soloid&mitid_loa_value=https%3A%2F%2Fdata.gov.dk%2Fconcept%2Fcore%2Fnsis%2FSubstantial). The MitID user must be present in the MitID pp environment. Custom theming is available and is used in this test flow. 

## Supported OIDC parameters

| **Request parameter** | **Description** |
| --- | --- |
| idp_values | **soloid** |


### Supported scope values

<table><tbody><tr><th><p><strong>Scope</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>soloid</p></td><td><p>List of claims:</p><ul><li>soloid.mitid_uuid</li></ul></td></tr></tbody></table>


## ID token claim values

<table><tbody><tr><th><p><strong>Claim</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>loa</p></td><td><p>Level of Assurance</p><p>One of</p><ul><li>https://data.gov.dk/concept/core/nsis/Low</li><li>https://data.gov.dk/concept/core/nsis/Substantial</li><li>https://data.gov.dk/concept/core/nsis/High</li></ul></td></tr><tr><td><p>amr</p></td><td><p>Authentication Method method used by the user</p><p>One of</p><ul><li>passkey</li><li>mitid</li><li>soloidauthenticator</li></ul></td></tr></tbody></table>

## Userinfo endpoint claim values for soloid scope

<table><tbody><tr><th><p><strong>Claim</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>soloid.mitid_uuid</p></td><td><p>The unique MitID identifier of the subject.</p></td></tr></tbody></table>

