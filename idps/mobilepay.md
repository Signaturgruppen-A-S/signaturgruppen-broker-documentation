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
