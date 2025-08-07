# Signaturgruppen Broker – AI Optimized Integration Reference

(Work in progress - testing)

This document provides a simplified and developer-friendly overview of how to integrate with Signaturgruppen Broker for MitID authentication.

## 🔐 MitID Authentication Flow

### 1. Redirect user to authentication

Initiate login by redirecting the user to Signaturgruppen’s authorization endpoint in PreProduction (PP):

```
GET https://pp.netseidbroker.dk/op/connect/authorize
```

With the following parameters:

- `client_id`: shared or issued client ID
- `redirect_uri`: your registered redirect URI
- `response_type=code`
- `scope=openid mitid`
- `state`: anti-forgery token
- `nonce`: for replay protection

---

### 2. User authenticates using MitID

The user is guided through the MitID login process via the MitID app or other method. On success, the user is redirected to your `redirect_uri` with:

- `code`
- `state`

---

### 3. Exchange code for tokens

To exchange the `code` for tokens, call the token endpoint:

```
POST https://pp.netseidbroker.dk/op/connect/token
```

With `application/x-www-form-urlencoded` payload:

- `grant_type=authorization_code`
- `code`: the authorization code received
- `redirect_uri`: same as above
- `client_id`
- `client_secret`

#### Example token response:
```json
{
  "access_token": "...",
  "id_token": "...",
  "refresh_token": "...",
  "expires_in": 300,
  "token_type": "Bearer"
}
```

Make sure to validate the `id_token` (JWT) properly.

---

## 📌 Tips & Support

### Get started for free

Customers can set up their first MitID login flows in **PreProduction (PP)** completely free of charge — no need for dedicated credentials initially.

Signaturgruppen provides shared `client_id` and `client_secret` values that can be used directly for testing purposes in PP.  
This allows you to quickly begin integration and try out the login flow without needing a prior agreement or custom configuration.

### When you're ready for dedicated credentials

Once you're ready to use your own credentials — whether in **PreProduction (PP)** or **Production (PROD)** — you can contact Signaturgruppen to have them issued.

📧 **Contact:** support@signaturgruppen.dk  
Please include your CVR number and a brief description of your setup or test environment.

Signaturgruppen's support team will assist you in obtaining your own `client_id` and `client_secret`, enabling you to move toward production or more realistic test scenarios.

---

## 🔗 Related Documentation

- [Signaturgruppen Broker – Full Documentation](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/)
- [OpenID Connect Spec](https://openid.net/specs/openid-connect-core-1_0.html)
