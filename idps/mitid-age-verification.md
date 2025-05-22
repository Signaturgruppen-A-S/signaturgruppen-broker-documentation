---
title: MitID Age verification
layout: home
parent: MitID
grand_parent: Identity providers
nav_order: 10
---

# MitID Age Verification
Minimal userdata and GDPR age verification using your Service Provider MitID agreement.

## Usage and parameters
Signaturgruppen Broker supports a userdata minimal, GDPR optimized age verification flow that utilizes MitID using the service providers active MitID agreement (logo, service provider name etc) with optimized idp paramaters such as MitID reference text and login flow AAL level defaults.

In order to active the flow, you setup an OIDC request as normal for MitID flows, with the following changes:

* **scope**: openid minimal age_verify:age
* **idp_values**: mitid (optinal, if MitID is the only active IdP for the client)

Example request: 
```
https://pp.netseidbroker.dk/op/connect/authorize?client_id=[your_client_id]&scope=openid minimal age_verify:18&response_type=code&idp_values=mitid&redirect_uri=https://yourdomain.dk/handle-mitid-age-verify&nonce=random-nonce-recommended
```

### ID token result
This will result in an ID token without any global/persistent values, which includes the following age verification claim:

```
idbrokerdk_age_verified: "18:true"
```

### Userinfo endpoint claims
This will result in a claim response from the Userinfo endpoint without any global/persistent values, which includes the following age verification claim:

```
idbrokerdk_age_verify_18: "true"
```

## Default Idp parameters
When the flow is started with the three scopes: 
```
scope:openid minimal age_verify:age
```
Then a set of specific age verification default parameters will be used for the flow. 

The default settings are
* A default MitID reference text appropriate for the age verification
* AAL level requested is set to NSIS level LOW. This is to allow for simplest MitID login for the end user.
* MitID "action text" is set to "CONFIRM", which informs the MitID client to word the flows as a "confirm" flow instead of "login".

## Minimal scope
The **minimal** scope value ensures that no global/persistent values are returned in terms of userdata through the flow **by default**. Only specifically requested claim values by specific scope values can then add persistent claim outputs. 

In terms of the age verification flow, then this is required in order to trigger the default settings, IdP parameters, text, AAL level etc. If **minimal** is not specified as a scope value, then the flow will still work, but will simply just add the requested claims, like the age verification claims if **age_verify:age** is specified.

## Override IdP parameters
If you specify the **idp_params** OIDC request parameter, this will override any setting towards MitID including the reference text and the requested AAL level. If **idp_params** is set, the flow will use default MitID flow values for any unspecified value and not use any of the age verification defaults.
