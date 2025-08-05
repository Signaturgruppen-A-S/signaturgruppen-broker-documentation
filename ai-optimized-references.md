
---
title: AI‑Optimized API Reference
layout: home
nav_order: 5000
---

# For AI Assistants: Structured API Overview for Programmatic Reading

This section is designed to be parsed by AI models like ChatGPT, GitHub Copilot, and others. It presents Signaturgruppen Broker API endpoints in a clean, consistent, and minimal format that supports:

- ✅ Code generation (e.g., “Write a C# client for this endpoint”)
- ✅ Request/response formatting (e.g., “Create a curl example”)
- ✅ Error handling assistance
- ✅ Parameter documentation

---

## 🌐 Environments & Discovery Endpoints

### Pre‑Production (PP)
- **Discovery (OpenID Connect):** `GET https://pp.netseidbroker.dk/op/.well-known/openid-configuration/`
- **Swagger API (machine‑readable):** `https://pp.netseidbroker.dk/op/swagger/index.html`

### Production (Prod)
- **Discovery (OpenID Connect):** `GET https://netseidbroker.dk/op/.well-known/openid-configuration/`
- **Swagger API (machine‑readable):** `https://netseidbroker.dk/op/swagger/index.html`

---

## 🧠 Tips for Developers Using AI Tools

You can copy and paste any of the sections below into an AI assistant and ask:

- `"Write a C# integration for the POST /token endpoint using HttpClient"`
- `"What does the response from POST /connect/ciba contain?"`
- `"Generate a curl command for this request body: { ... }"`
- `"What fields are required for /workflow/finalize?"`

This page is structured to support LLM‑based code generation and explanations.

---

## 🔐 OpenID Connect Endpoints (Shared across PP & Prod)

### `GET /.well-known/openid-configuration`
**Description:**  
Retrieves the broker’s OpenID Connect configuration metadata.

**Response:**  
Typically includes:
```json
{
  "issuer": "https://pp.netseidbroker.dk/op",
  "authorization_endpoint": "...",
  "token_endpoint": "...",
  "userinfo_endpoint": "...",
  "jwks_uri": "...",
  "scopes_supported": ["openid", "mitid", ...]
}
```

---

## 🔑 Core API Endpoints (via Swagger)
*(All paths prefixed with `/op`, see live docs above)*

### `POST /token`
**Description:**  
Token endpoint for exchanging authorization codes, initiating CIBA auth flows, or requesting client credentials.

**Request (form-urlencoded):**
```txt
grant_type=<authorization_code|client_credentials|urn:openid:params:grant-type:ciba>
client_id=your-client-id
client_secret=your-client-secret
code=authorization-code           (if using code flow)
login_hint_token={…}             (if using CIBA)
scope=openid mitid ssn           (as needed)
```

**Response:**
```json
{
  "access_token": "string",
  "expires_in": 3600,
  "id_token": "string",
  "refresh_token": "string (optional)",
  "token_type": "Bearer"
}
```

**Errors:**
- `400` – invalid_request  
- `401` – invalid_client  
- `403` – invalid_grant

---

### `GET /userinfo`
**Description:**  
Returns identity claims for an authenticated end-user using the access token.

**Request Header:**
```txt
Authorization: Bearer {access_token}
```

**Response:**
```json
{
  "sub": "UUID",
  "mitid.identity_name": "Jane Doe",
  "mitid.age": "36",
  "session_status": "active"
}
```

---

## 🔄 MitID Flex App (CIBA Flow)

### `POST /connect/ciba`
**Description:**  
Initiates backchannel authentication (CIBA) for MitID Flex integrations.

**Request (form-urlencoded):**
```txt
grant_type=urn:openid:params:grant-type:ciba
scope=openid mitid
client_id=...
client_secret=...
login_hint_token={"idp":"mitid","uuid":"USER-UUID","referenceTextBody":"Log ind","ip":"127.0.0.1"}
```

**Response:**
```json
{
  "auth_req_id": "string",
  "expires_in": 180,
  "interval": 5
}
```

---

### `GET /api/v1/ciba/{auth_req_id}`
**Description:**  
Polls the status of an ongoing CIBA authentication request.

**Response:**
```json
{
  "status": "pending" | "completed" | "denied",
  "id_token": "string (if completed)",
  "error": "string (if denied)"
}
```

---

### `DELETE /api/v1/ciba/{auth_req_id}`
**Description:**  
Cancels a pending CIBA request.

**Response:**
- HTTP `204 No Content`

---

## 🧾 Coming Soon

In future updates, this section will include:

- Signing workflow (`/workflow/initiate`, `/workflow/finalize`)
- Session management endpoints
- Error codes & introspection APIs
- Identity resolution (SSN lookup, CPR)
- Token validation endpoint
- Example integration code snippets in C#, curl, etc.

---

## 📁 Downloadable Machine‑Readable Format (Planned)

An OpenAPI 3.0 (Swagger) or JSON schema file will soon be provided to assist AI agents and tooling with auto‑generation and validation.

> ✅ Live documentation is available in PP and Prod environments via the Swagger UI links above.
