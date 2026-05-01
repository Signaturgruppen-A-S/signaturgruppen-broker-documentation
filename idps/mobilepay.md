---
title: MobilePay
layout: home
parent: Identity providers
has_children: false
nav_order: 10
---

# MobilePay Login
"A stress free sign-up and login experience for your customers".

MobilePay Login provides service providers with a consent-based login option with a frictionless user experience. 
Read more at [https://vippsmobilepay.com/da-DK/login](https://vippsmobilepay.com/da-DK/login).

Contact Signaturgruppen for access to MobilePay login.

## Setting up PP integration
To setup integration in PP, ask our sales/support for opening for MobilePay and provide us with the following details:

### Setup merchant (business) in the MobilePay portal
First, you will need to setup a merchant in the MobilePay portal [https://vippsmobilepay.com/en-DK/login](https://vippsmobilepay.com/en-DK/login). 
Then you can setup your test and production details for MobilePay login. 

When setup. Click the "For developers" in the bottom left corner, then select test and then "setup login":

<img width="2066" height="1137" alt="mobilepay_portal_setup_udviklere" src="https://github.com/user-attachments/assets/1a4de078-0d28-4fa5-afbf-0040e6f64554" />

Next, setup the following redirect URIs:

```
https://pp.netseidbroker.dk/op/signin-oidc-dynamic-mobilepay
```

```
https://idbroker.eu/op/signin-oidc-dynamic-mobilepay
```

Set token endpoint method to **"client_secret_post"**.

<img width="1212" height="1201" alt="mobilepay_portal_setup_1" src="https://github.com/user-attachments/assets/ea87e55e-e385-4f35-812e-bee93169fecb" />

Then consider your pricing tier model, depending on what data you want to have returned:

<img width="677" height="717" alt="mobilepay_portal_pricing" src="https://github.com/user-attachments/assets/2bab6983-6600-4754-af19-ac5a233938a2" />

Last, from the "developers", under test, you can go to "show keys". From here either send client_id and client_secret to Signaturgruppen or request access to the Signaturgruppen Broker Admin portal, where you will be able to enter these details and setup your integration.

## MobilePay testing
In the MobilePay portal [https://vippsmobilepay.com/en-DK/login](https://vippsmobilepay.com/en-DK/login), you can setup your own test merchant and then setup test users and get info on their test app. This can be used in our PP environment.

## Demo
A test of MobilePay integration login and flow can be tested via: https://brokerdemo-pp.signaturgruppen.dk/ (quick start or advanced variants).
This will also allow for analysing Signaturgruppen Broker returned tokens and claims.

## OIDC integration
Signaturgruppen Broker supports the same scopes and claims for the MobilePay Login flow, as defined by MobilePay in: [https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/core-concepts/](https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/core-concepts/).

Supported MobilePay scopes: [https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/user-info/#scopes](https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/user-info/#scopes)

### NIN (national identity number)
MobilePay Login supports handout of user NIN (CPR). This requires user consent, just as any other requested information, and requires the merchant to be at the advanced pricing tier.

### MobilePay Step-up
Default for MobilePay Login is a frictionless login experience, which allows the user to "remember" the browser used and reuse the authentication in that browser for subsequent logins. If the service provider requires a new MobilePay App authentication, the "step-up" flow can be utilized. 
To invoke the "step-up" flow, either
* Configure prompt=login to trigger MobilePay "step-up" in the Signaturgruppen Broker Admin portal
* Set the acr_values="urn:vipps:acr:app_auth" in the OIDC request.

See MobilePay docs: [https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/step-up-authentication/](https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/step-up-authentication/)

This requires the merchant to be at the advanced pricing tier.

### Example transaction_token :
The optional transaction token demonstrates expected claims.

```
{
  "birthdate": "19..3",
  "address": "{\"street_address\":\"Weldon Trafficway 48w\\ne\",\"formatted\":\"Weldon Trafficway 48w\\ne\\n4267\\nSouth Jazminchester\\nDK\",\"region\":\"South Jazminchester\",\"postal_code\":\"4267\",\"country\":\"DK\",\"address_type\":\"home\"}",
  "gender": "male",
  "given_name": "Lacey",
  "name": "Lacey Walter",
  "phone_number": "45..19",
  "family_name": "Walter",
  "email": "b...m",
  "idp": "mobilepay",
  "idp_identity_id": "c2fa...ed",
  "auth_time": "1777626074",
  "sub": "c2fa...ed",
  "transaction_id": "eeab..a",
  "redirect_uri": "https://brokerdemo-pp.signaturgruppen.dk/signin-oidc",
  "transaction_actions": "mobilepay_authenticated",
  "transaction_client_ip": "1....9",
  "reference_id": "demo_normal_flow_requestid",
  "nbf": 1777626075,
  "exp": 1967014875,
  "iat": 1777626075,
  "nin": 2......4
  "iss": "https://pp.netseidbroker.dk/op",
  "aud": "14...36"
}
```


## Setting up Prod integration
We are working on a partner setup with MobilePay which allows for easy integration to the production environment, without the need of handling technical details such as URLs and secrets.

Currently the process for production access is the same as for the PP environment.

### Redirect URIs
```
https://netseidbroker.dk/op/signin-oidc-dynamic-mobilepay
```

```
https://idbroker.eu/op/signin-oidc-dynamic-mobilepay
```
