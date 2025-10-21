---
title: Microsoft Entra ID
layout: home
parent: Enterprise features
nav_order: 25
---

# Microsoft Entra ID Integration
Signaturgruppen Broker supports integrations from Microsoft Entra ID as an external identity provider.

## Documentation reference
The integration guide can be found via the Microsoft documentation online at: 

[https://learn.microsoft.com/en-us/entra/external-id/customers/how-to-custom-oidc-federation-customers](https://learn.microsoft.com/en-us/entra/external-id/customers/how-to-custom-oidc-federation-customers)

This will guide you through the basic OpenID Connect setup such as configuring redirect url, authority, claim mappings, scope usage etc.

## Claim mappings
Various claim mappings are supported by Microsoft Entra ID. See the specific claims from the utilized identity providers used via Signaturgruppen Broker in order to list the relevant claims to map into your Entra ID setup.

## Force email (required)
Microsoft Entra ID requires an email claim from any external identity provider, however this claim is not part of many IdPs available via the Signaturgruppen Broker platform such as MitID.
In cases with identities that does not have an email claim, the special scope **force_email** is supported, which ensures that an email claim is returned as part of the result. If no real email claim was given by the chosen eID/IdP, then the email will have the form of 

```
[unique-idp-ID]@email.invalid
```

See [https://www.rfc-editor.org/rfc/rfc6761#section-6.4](https://www.rfc-editor.org/rfc/rfc6761#section-6.4) for reference on the use of **.invalid**.

## Missing any information?
If you miss any information to help your integration to Signaturgruppen Broker from Microsoft Entra ID, please reach out to our support at support.signaturgruppen@ingroupe.com.
