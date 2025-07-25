---
title: Ciba integration
layout: home
parent: MitID Flex app
grand_parent: Enterprise features
nav_order: 2
---

## Ciba integration
Signaturgruppen Broker has implemented the MitID Flexibility app (addon) using the Client Initiated Backchannel Authentication specification [CIBA](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/references.html#ciba). 

This section describes the technical details needed in order to integrate to the setup.

## Getting started
To familiarize yourself with the setup, it is recommended to start out using our open and free demo setup for the PP environment, which includes

* **A preconfigured demo service provider**: Approved for Flex app and approved for no channel binding.
* **Demo client credentials**: Preconfigured and openly available **client_id** and **client_secret**.

```js
service provider name: "Signaturgruppen flex app demo"
client_id: "62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9"
client_secret: "mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ=="
```

### Postman example

The test example is also available as a [Postman collection here](https://raw.githubusercontent.com/Signaturgruppen-A-S/signaturgruppen-broker-documentation/main/enterprise/files/ciba_pp.postman_collection.json)

## Steps of the flow (API)

### Initiating

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid mitid' \
--data-urlencode 'client_id=62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9' \
--data-urlencode 'client_secret=mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ==' \
--data-urlencode 'login_hint_token={"idp":"mitid", "uuid":"8a9856e0-f12d-4217-b320-c8a076be9320", "referenceTextBody":"Testing MitID Flex app 😉", "ip":"1.1.1.1"}'
```

The login_hint_token can also be Base64 encoded:
```
--data-urlencode 'login_hint_token=eyJpZHAiOiJtaXRpZCIsImlkcF9wYXJhbXMiOiJ7InV1aWQiOiI4YTk4NTZlMC1mMTJkLTQyMTctYjMyMC1jOGEwNzZiZTkzMjAiLCAicmVmZXJlbmNlVGV4dEJvZHkiOiJUZXN0aW5nIE1pdElEIEZsZXggYXBwIPCfmIkiLCAiaXAiOiIxLjEuMS4xIn0ifQ=='
```

This would result in a response on the following form

```
{
    "auth_req_id": "0A2179A15B686CB..3AC29A5A6A-1",
    "expires_in": 300,
    "interval": 5
}
```

### Poll
Using the **auth_req_id** received in the initiation response, you can then poll for status. 

> Note that the **interval** in the init response specifies the required interval between polls in seconds.

```
curl --location 'https://pp.netseidbroker.dk/op/connect/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid' \
--data-urlencode 'client_id=62b8cc03-aa73-4d6a-922c-7c8fb0a4a1d9' \
--data-urlencode 'client_secret=mIpxk84FI1wpc7cP7nodFLfgQQ1ScZHYO44FZ9wPOqrB0ha9S5RUNYPXMkrCWwRjGqEH0hflnIJea8IKmW19aQ==' \
--data-urlencode 'auth_req_id=0A2179A15B686CB..3AC29A5A6A-1'
```

If the flow is still pending you will receive the following response:

```
{
    "error": "authorization_pending"
}
```

When the flow has completed successfully (example):

```
{
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJKV1QifQ.eyJpc3MiOiJodHRwczovL3BwLm5ldHNlaWRicm9rZXIuZGsvb3AiLCJuYmYiOjE3MTUwODc0NTMsImlhdCI6MTcxNTA4NzQ1MywiZXhwIjoxNzE1MDg3NzUzLCJhdWQiOiI2MmI4Y2MwMy1hYTczLTRkNmEtOTIyYy03YzhmYjBhNGExZDkiLCJhdF9oYXNoIjoiaW9NTjhaVndrdWs5czdxNk1sc1NGQSIsInN1YiI6IjdjOTQ4NGY1LTQ0YmYtNGUyOS1hNjgzLTg0NTQ4OTUxZDNiOCIsImF1dGhfdGltZSI6MTcxNTA4NzQ1MywiaWRwIjoibWl0aWQiLCJtaXRpZC51dWlkIjoiZWVlNGIyNzItNmEzMy00MGRhLWFiMGItNWNlMmFiZDQyZTc4IiwibWl0aWQuYWdlIjoiODgiLCJtaXRpZC5kYXRlX29mX2JpcnRoIjoiMTkzNi0wNC0wOCIsIm1pdGlkLmhhc19jcHIiOiJ0cnVlIiwibWl0aWQuaWRlbnRpdHlfbmFtZSI6IklsZXlhIEplbnNlbiIsIm1pdGlkLnBzZDIiOiJmYWxzZSIsIm1pdGlkLnJlZmVyZW5jZV90ZXh0Ijoib3N0ZW1hd2QgOj0pIiwibWl0aWQudHJhbnNhY3Rpb25faWQiOiI5NTgxYmFiNi05ZjhhLTQ1MjYtOWRkYy1kZmQzOTgxMjYzYjkiLCJpZHBfdHJhbnNhY3Rpb25faWQiOiI5NTgxYmFiNi05ZjhhLTQ1MjYtOWRkYy1kZmQzOTgxMjYzYjkiLCJpZHRva2VuX3R5cGUiOiJjaWJhIn0.FINs1cD2vMnnifJ0ccHi7GgMD1m4u8uZAqbVnGLncYbx3FUuW7Hl64VLQj8IAABXv8jGlX1BoYh1dKhNaJTmka-VKhR_rh1hAFrEInFvS4DYX0HKk5mVm1VOMdGcl2dzKfAjEVDq1tGcCr-HPWA7DR3Mt4vbwWJ5-ZOEJ0mA-49YZsCwNB855dcNf3PdmI0mCknUPmHLf4-ZSIfda4_tW2AmHAmJWtfWUPEQDGMLbEqkbggkETy50J06y9NM4GbczT035PW9EzeJjrqjzvgVtSy3eizRbl2W0IR95hDV1UoJTqW9xn8pt6Qs897TKbjXAmjOPphJ_jE1uGozId1RiQ",
    "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IjA0ODA1OEJCNTlGNEQzMDA3MDQ1ODk2RkQ0ODhDRTgxRjRFQjQ5MjMiLCJ0eXAiOiJhdCtqd3QifQ.eyJpc3MiOiJodHRwczovL3BwLm5ldHNlaWRicm9rZXIuZGsvb3AiLCJuYmYiOjE3MTUwODc0NTMsImlhdCI6MTcxNTA4NzQ1MywiZXhwIjoxNzE1MDkxMDUzLCJzY29wZSI6WyJvcGVuaWQiLCJtaXRpZCJdLCJjbGllbnRfaWQiOiI2MmI4Y2MwMy1hYTczLTRkNmEtOTIyYy03YzhmYjBhNGExZDkiLCJzdWIiOiI3Yzk0ODRmNS00NGJmLTRlMjktYTY4My04NDU0ODk1MWQzYjgiLCJhdXRoX3RpbWUiOjE3MTUwODc0NTMsImlkcCI6Im1pdGlkIiwiaWRwX3RyYW5zYWN0aW9uX2lkIjoiOTU4MWJhYjYtOWY4YS00NTI2LTlkZGMtZGZkMzk4MTI2M2I5IiwianRpIjoiNUIwODQ1QkREQUNGQkJERDgxMzFBNkJEOENEQkI1NDcifQ.qjU0PUBmTR90OLhm6CI6VitHr9MRVPicl2ukXBAFu5ZVI0zFTYtHKcU2yYcP5rosDEojffWdo4HxRrqx0UztGQT6RWaUBetMDv3VU0kodVJwkmmchgKL6_P7CMGnC0e2ktC-8inAiNRsPGT_P2j8p3g8cALXeSKjZKadF4e-NNpqrb37Od9cIR2otHv44IXKuUu2wxDfdeCKGYMS4pFbfB_n9aav8P1JADazX2TAkmE02N8M-J2XVFsBzzZ2G5_Us93KEYqIa2rRKGYaEWtQtYHvcfkNKor9LzB6DbIL0N3R4-IJipa2On6W9aJF6IbqnARe4PAC-iNeEtjZsx9k9A",
    "expires_in": 3600,
    "token_type": "Bearer",
    "scope": "openid mitid"
}
```

### Cancel
To cancel a flow, invoke the cancel API endpoint using the received **auth_req_id** received when initiating the flow.

```
curl --location --request DELETE 'https://pp.netseidbroker.dk/op/api/v1/ciba/0A2179A15B686CB..3AC29A5A6A-1' \
```

## MitID Appswitch without channel-binding
Appswitching to the MitID app can be done with or without channel-binding enabled. Currently we have only listed support for the variant without built-in channel-binding, but will add the channel-binding variant on-demand.

In order to activate MitID appswitching without channel-binding, the following steps must be taken.
* The integrating service must be exempted from the channel-binding requirement
* The MitID flow must be successfully started for the user.
* The integrating app should verify that the MitID app is installed on the running system. If not able to do so, there should be a strong indication of some means that the user would want to appswitch.
* Appswitch to the OS specific static MitID appswitch URL.
* Optionally setup a "returnUrl" query parameter in the MitID appswitch URL. This is used by MitID to redirect the end-user back to the integrating app.

### Testing for the MitID app

Testing the presence of the MitID app on the device can be done in the ways described below for each of the platforms.

#### Android
On Android, you can use explicit intents to switch to another app. We can use a similar approach to check, if an app is installed on the device. Below is a code snippet for how you can check, if the MitID app is installed on the device.
```
public boolean deviceHasMitIDApp() {
    try {
        getPackageManager().getPackageInfo("dk.mitid.app.android", 0);
        return true;
    } catch (PackageManager.NameNotFoundException e) {
        return false;
    }
}
```

On Android 11, Google changed the package visibility for apps, meaning that checking for the presence of an app based on package name requires a bit more setup. In your AndroidManifest.xml, you must add which package names you want to be able to query about. The code snippet below shows how to add the MitID app to your queries in the manifest file.
```
<manifest …>
    <queries>
        <package android:name="dk.mitid.app.android" />
    </queries>

    <application …. />
</manifest>
```

#### iOS
On iOS, to check if the MitID app is installed on the device, the service provider app can query for the URLScheme of the MitID app (“mitid-app”). The SP app must declare its intent to query for this scheme by adding the URLScheme “mitid-app” to its Info.plist file under the key LSApplicationQueriesSchemes. See: [IOSCANOPEN]. The SP app can then check for the presence of the MitID app on the device as detailed below:
```
func canOpenMitIDApp() -> Bool {
    guard let url = URL(string: "mitid-app://") else {
        return false
    }
    return UIApplication.shared.canOpenURL(url)
}
```
### MitID static appswitch URL
In order to appswitch (without channel-binding) to the MitID app, (appswitch-) redirect to the following OS specific URL.

#### Android
> Important for the Android URL to URL encode it as shown in the examples.

> NOTE, changes with version 15 of MitID for Android. Following documentation is for version 15, which is live in PP and will become live in production 19.08.2025

All environments (Install the MitID app for target environment last on device, this will handle the appswitch):
```
https://appswitch.mitid.dk
```
With **returnUrl** parameter (requires trailing **"/"** on MitID URL):
```
https://appswitch.mitid.dk/?returnUrl=https://appswitch.to.app/android
```

#### IOS
PP environment:
```
https://appswitch-test.mitid.nets.eu
```

Production environment:
```
https://appswitch.mitid.dk
```

With **returnUrl** parameter (requires trailing **"/"** on MitID URL):
```
https://appswitch.mitid.dk/?returnUrl=https://appswitch.to.app/ios
```
## Status codes from Poll
When invoking the poll CIBA endpoint, non completed/successful results will be returned in the following JSON format, here with the "pending" example:

```
{
    "error": "authorization_pending"
}
```

In the following the different error codes will be mapped. These errors are returned with HTTP status code 400 Bad Request. 

| Error | Description |
|--------|--------|
|  authorization_pending      | Flow still pending accept/reject from end-user       |
|  invalid_grant      | Invalid request id or flow no longer valid. Note that tokens can only be returned once, then flow is invalidated.      |
|  access_denied      | End-user has rejected the authentication request.      |
|  slow_down      | Request is still pending, but polling is too frequent.     |
|  expired_token      | The request_id has expired     |
| mitid_flex_app.mitid_error_{mitid_status_code} | Error from MitID with the corresponding MitID status code |

## Signed request object
We support sending requests as a signed request object. This object is sent as a signed JWT.

The following code is a simple example of how to construct a signed JWT in dotnet. The secret in this example is a shared secret but an assymetric key can be used as well. 
```
    public string CreateRequestObject(string opIssuer, string clientId, string clientSecret, IDictionary<string, object> parameters)
    {
        var now = DateTime.UtcNow;
        var securityTokenDesciptor = new SecurityTokenDescriptor
        {
            Issuer = clientId,
            Audience = opIssuer,
            IssuedAt = now,
            NotBefore = now,
            Expires = now.AddMinutes(20),
            Claims = parameters,
            SigningCredentials = GetSigningCredentials(clientSecret)
        }; 
        var jwtHandler = new JsonWebTokenHandler();
        return jwtHandler.CreateToken(securityTokenDesciptor);
    }

    private static SigningCredentials GetSigningCredentials(string clientSecret)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(clientSecret));
        return new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    }
```
The **opIssuer** value in this context is the issuer value from our discovery endpoint.

We can construct the parameters to the request object in the following way:
```
            var parameters = new Dictionary<string, object>
            {
                { "client_id", clientId },
                { "login_hint_token", loginHint },
                { "scope", "openid mitid" },
                { "jti", Guid.NewGuid() }
            };
```

Finally we can add it to the request and send it

```
            var formContent = new FormUrlEncodedContent(new[]
            {
                new KeyValuePair<string, string>("client_id", clientId),
                new KeyValuePair<string, string>("client_assertion", clientAssertion),
                new KeyValuePair<string, string>("client_assertion_type", "urn:ietf:params:oauth:client-assertion-type:jwt-bearer"),
                new KeyValuePair<string, string>("request", requestObject),
            });
            var response = await new HttpClient().PostAsync(new Uri(authority + "/connect/ciba"), formContent);
```

In the above example we also used a client assertion instead of sending the secret. Below is described how to construct the client assertion

## Client assertions
Instead of sending the client secret, we can send a client assertion. We can generate it using the following method:
```
    public string CreateClientAssertion(string opIssuer, string clientId, string clientSecret)
    {
        var now = DateTime.UtcNow;
        var securityTokenDesciptor = new SecurityTokenDescriptor
        {
            Issuer = clientId,
            Audience = opIssuer,
            IssuedAt = now,
            NotBefore = now,
            Expires = now.AddMinutes(20),
            Claims = new Dictionary<string, object>
            {
                { "jti", Guid.NewGuid() },
                { "sub", clientId },
            },
            SigningCredentials = GetSigningCredentials(clientSecret)
        };
        var jwtHandler = new JsonWebTokenHandler();
        return jwtHandler.CreateToken(securityTokenDesciptor);
    }

    private static SigningCredentials GetSigningCredentials(string clientSecret)
    {
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(clientSecret));
        return new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    }
```
The client assertion is then added to the request at the parameter: **client_assertion**. Further the **client_assertion_type** parameter must be set to **urn:ietf:params:oauth:client-assertion-type:jwt-bearer**
