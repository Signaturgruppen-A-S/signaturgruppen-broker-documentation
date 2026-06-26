---
title: MobilePay
layout: home
parent: Identity providers
has_children: false
nav_order: 20
---

# MobilePay Login
"A stress free sign-up and login experience for your customers".

MobilePay Login provides service providers with a consent-based login option with a frictionless user experience. 
Read more at [https://vippsmobilepay.com/da-DK/login](https://vippsmobilepay.com/da-DK/login).

<img width="2600" height="1500" alt="image" src="https://github.com/user-attachments/assets/9741b2ac-61a8-4aef-bd79-0c6e091d7127" />

Contact Signaturgruppen for access to MobilePay login.

## Setting up MobilePay integration

* Setup your sales unit / merchant in the MobilePay portal: https://developer.vippsmobilepay.com/docs/knowledge-base/applying-for-services/.
* For production setup, ask our sales/support for help enabling IN Groupe Signaturgruppen A/S as your technical Partner for MobilePay Login.
* For PP setup, see section at the bottom of this documentation.

As soon as IN Groupe Signaturgruppen A/S has been setup as the technical partner within the MobilePay administration for your registered merchant unit, you can enable MobilePay Login for your services in Signaturgruppen Broker Admin UI.

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

### Custom Flow
To setup your integration to utilize the [MobilePay Custom Flow](https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/custom-flow-migration/), specify the additional scope: **mobilepay_customFlow**.

Example scope request: 
```
scope=openid name phoneNumber mobilepay_customFlow
```

This will trigger your configured Custom Flow and result in the additional **mobilepay_flowResult** claim in both Userinfo Endpoint and Transaction claims (transaction token).

Example **mobilepay_flowResult** claim:
```
{
  "flowResult": {
    "language": "EN",
    "screens": [
      {
        "title": "Extra consent",
        "description": "Accept terms and conditions",
        "terms": {
          "links": [
            {
              "title": "Terms of Service",
              "url": "https://example.com/terms"
            }
          ]
        },
        "elements": [
          {
            "type": "checkbox",
            "id": "cf_accept_terms",
            "title": "Accept T&C",
            "checked": true,
            "required": true,
            "readMore": {
              "title": "Read more",
              "description": "Details about the custom flow"
            }
          }
        ]
      }
    ],
    "timeOfConsent": "2026-01-17T09:31:00Z"
  }
}
```

### MobilePay Step-up
Default for MobilePay Login is a frictionless login experience, which allows the user to "remember" the browser used and reuse the authentication in that browser for subsequent logins. If the service provider requires a new MobilePay App authentication, the "step-up" flow can be utilized. 
To invoke the "step-up" flow, either
* Configure prompt=login to trigger MobilePay "step-up" in the Signaturgruppen Broker Admin portal
* Set the acr_values="urn:vipps:acr:app_auth" in the OIDC request.

See MobilePay docs: [https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/step-up-authentication/](https://developer.vippsmobilepay.com/docs/APIs/login-api/api-guide/step-up-authentication/)

This requires the merchant to be at the advanced pricing tier.

### Example tokens :
ID token:

```
{
  "iss": "https://idbroker.eu/op",
  "nbf": 1781809378,
  "iat": 1781809378,
  "exp": 1781809678,
  "aud": "e..a",
  "nonce": "639174061751992403.YzE3NTlmYjgtMWVmMi00YzlhLTk3YzQtODdlYmIxMDBlMjQ1ZGZhNDhhNGYtOThiMS00MWVjLTgyNGQtY2U4ZWUwMzRmNjUx",
  "at_hash": "6I60BHbq1XQyXzVZBkV07A",
  "sid": "ef22a6bc-ef47-460b-8912-201572f8761f",
  "sub": "49..5b54",
  "auth_time": 1781809383,
  "idp": "mobilepay",
  "neb_sid": "ef22a6bc-ef47-460b-8912-201572f8761f",
  "transaction_id": "124b9076-9a57-4b0a-8557-43e1076c8703",
  "session_expiry": "1781823778",
  "mobilepay_msn": "2..1",
  "mobilepay_client_id": "b..2",
  "subject_type": "idp_id",
  "rat": "1781809383"
}
```

The optional transaction token demonstrates expected claims. (Standard integrations use ID token + Userinfo endpoint to retrieve relevant claims).
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
  "mobilepay_msn": "20..1",
  "mobilepay_client_id": "a..d471"
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
  "iss": "https://idbroker.eu/op",
  "aud": "14...36",

}
```

## Setting up PP integration
To setup integration in PP, ask our sales/support for opening for MobilePay for PP and provide us with the following details:

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
https://pp.idbroker.eu/op/signin-oidc-dynamic-mobilepay
```

Set token endpoint method to **"client_secret_post"**.

<img width="1212" height="1201" alt="mobilepay_portal_setup_1" src="https://github.com/user-attachments/assets/ea87e55e-e385-4f35-812e-bee93169fecb" />

Then consider your pricing tier model, depending on what data you want to have returned:

<img width="677" height="717" alt="mobilepay_portal_pricing" src="https://github.com/user-attachments/assets/2bab6983-6600-4754-af19-ac5a233938a2" />

Last, from the "developers", under test, you can go to "show keys". From here either send client_id and client_secret to Signaturgruppen or request access to the Signaturgruppen Broker Admin portal, where you will be able to enter these details and setup your integration.

