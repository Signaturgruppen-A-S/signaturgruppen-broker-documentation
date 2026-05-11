---
title: e-Boks ID
layout: home
parent: Identity providers
has_children: false
nav_order: 19
---

# e-Boks ID
[e-Boks ID ](https://corporate.e-boks.com/loesninger/e-wallet/e-boks-id/) is an ID integrated directly into your e-Boks app and allows you to login and share information such as your name, age and national identity number to service providers.

<img width="1068" height="980" alt="image" src="https://github.com/user-attachments/assets/86b3f1f3-1a5a-4ee1-bc19-ed09b4fe7063" />

## OIDC integration

We have added 3 scopes that applies to e-Boks ID flows via Signaturgruppen Broker

```
eboks -> eboks.identity_id + eboks.name
```
```
eboks_info -> eboks.date_of_birth
```
```
eboks_national_id -> eboks.national_id
```

Additional claim mappings will be added as additional information become available from e-Boks ID, such as phone number and email.

These represent a basic level of claims for just the identity with the **eboks** scope. Then a request for additional information on the user via the the **eboks_info** scope.
The **eboks_national_id** scope requests the eboks national identity (CPR) claim.
 
### Supported identity provider parameters (idp_params -> eboks)

<table>
    <tbody>
        <tr>
            <th>
                <p><strong>Identity Provider parameters (eboks)</strong></p>
            </th>
            <th>
                <p><strong>Description</strong></p>
            </th>
        </tr>
     <tr>
            <td><p>purpose</p></td>
            <td>
             <p>Type: String.</p>
             <p>This text will be displayed as part of the approval screen in the e-Boks app</p>
             <p>Defaults to "Approve"/"Godkend".</p>
            </td>
        </tr>
        <tr>
            <td>
                <p>qr_message</p>
            </td>
            <td>
             <p>Type: String.</p>
             <p>This text will be displayed as "action text" at the QR code / app-switch step.</p>
             <p>Defaults to the same value as <em>purpose</em></p>
            </td>
        </tr>
        <tr>
            <td>
             <p>reference_text</p>
            </td>
            <td>
                <p>Type: String</p>
                <p>If present, this message will be shown to the end-user in the e-Boks app as part of approval.</p>
            </td>
        </tr>
        <tr>
            <td>
                <p>escalation_level</p>
            </td>
            <td>
                <p>Type: String enum</p>
                <p>One of: "low", "medium" or "high"</p>
            </td>
        </tr>
        <tr>
            <td><p>return_uri</p></td>
            <td>
                <p>Type: string (URI)</p>
                <p>if set, the e-Boks app will redirect the end-user to the URI after approval in the e-Boks app.</p>
            </td>
        </tr>
        
    </tbody>
</table>


### Example JSON for identity providers

```json
{
  "eboks": {
    "reference_text": "Approve handout of data"
  }
}
```

```
idp_params=%7B%20%22eboks%22%3A%7B%22Approve%20handout%20of%20data%22%3A%22%22%7D%7D
```


## Test (PP) 

[Signaturgruppen Broker PP Demo](https://brokerdemo-pp.signaturgruppen.dk/) includes the option to test the e-Boks ID in a test environment. This requires download of the e-Boks ID test app and setting this up with a valid MitID test user. 
Reach out to Signaturgruppen for more information on MitID test users.

[Signaturgruppen Broker Demo (production)](https://brokerdemo.signaturgruppen.dk/) includes the option to test the e-Boks ID with your own production e-Boks App.

### Android test app (Open URL on your Android device)
On Android you can setup the test app via Firebase: [https://appdistribution.firebase.dev/i/e11a71fa1347809f](https://appdistribution.firebase.dev/i/e11a71fa1347809f)

### iOS test app (Open URL on your iOS device)
On iOS you can setup the test app via TestFlight: [https://testflight.apple.com/join/FH8k2nkh](https://testflight.apple.com/join/FH8k2nkh)

## Production
e-Boks ID is generally available as an option via the integration to Signaturgruppen Broker. It must be enabled for your organization, and then it will become available as option for your service providers and clients via Signaturgruppen Broker Admin UI.
