---
title: MitID Flex app
layout: home
parent: Enterprise features
nav_order: 5
has_children: true
---

# MitID Flexibility app
MitID Flexibity app is the possibility to utilize the MitID app as the only authenticator for flows where a standard browser is not available. 
The usecases includes applications on smart-TVs, over-the-phone support and help-desk scenarios, legacy hardware terminals etc. In these cases, a normal, secure and up-to-date browser is not available, but a valid usecase for MitID is found. 

Contact Signaturgruppen for more information.

> Please note that this version of MitID is not generally available and must be accepted and certified for each specific use case.

## Client Initiated Backchannel Authentication
The MitID Flex app variant has been implemented using the Client Initiated Backchannel Authentication (CIBA) specification. 
A general overview of this protocol is outlined here:

Integrating to CIBA involves several technical steps designed to ensure a secure and efficient authentication process. Here's an overview of the steps involved:

1. **Client Registration**: Use our free demo client or contact Signaturgruppen for your own client.

3. **Authentication Request Initiation**:
    - The client application initiates an authentication request to the authentication server, providing necessary information about the end-user that is to be authenticated.
    - This request includes parameters like `scope`, `client_id`, and the `login_hint` to identify the user.

4. **User Notification**:
    - The authentication server processes the request and, if the user is recognized, sends a notification to the user’s device via an out-of-band method (e.g., a mobile app push notification).

5. **End-User Authentication**:
    - The user receives the notification and performs authentication on their device, which could involve biometrics, a PIN, or another form of user verification.

6. **Token Issuance**:
    - Upon successful user authentication, Signaturgruppen Broker sends a response back to the client. This requires the client to poll the server for the token.
    - The server issues tokens (like ID token and access token) following the successful authentication.

These steps ensure that the authentication process is user-friendly, secure, and adheres to the OpenID Connect standards which underpin the CIBA flow. This setup is particularly useful for decoupling device scenarios where the device requesting authentication is not the same as the user's device, allowing for more flexible authentication methods.

# Online demo example
We have constructed an [online demo](https://brokerdemo-pp.signaturgruppen.dk/flexapp) showcasing a standard phone helpdesk usecase. 

MitID Flex app is by design a backchannel protocol, which enables services to initiate the MitID app authenticator directly using a backend to backend API published by Signaturgruppen Broker. 

It is important to note, that this is just one possible usecase. As a guiding principle, the flow should mimic the real MitID flow in as much as possible, in terms of UX, behavior and user information, but as showcased in the demo it is possible to have a complete UI-less MitID experience, if no UI rendering is possible.

