---
title: Broker APIs
layout: home
nav_order: 10
---

## Signaturgruppen Broker API
APIs are provided as REST APIs, available over HTTPS (HTTP/1.1 protected by TLS 1.2 or higher).

### Swagger endpoint

Swagger definition endpoint URL
```
[Authority URL]/swagger/index.html
```

### API specification
All APIs are specified according to the OpenAPI 3.0 specification (previously known as “Swagger”). In practice this means that the APIs are described in machine-readable YAML documents, describing the resources exposed in the APIs, the available methods etc. as well as human-readable descriptions of the API. The YAML files can then be used to create API-specific clients asnd stubs or be used as input into tools for API testing. They can also be used for generating documentation. The API documentation is delivered in the form of HTML files generated from the YAML specifications

### Error Handling
Errors are reported using HTTP error codes. Each API function documents the error codes that may be re-turned. In addition, a JSON error object is returned that provides further information on the error. The structure of this error object is described in the YAML files for the specific API.

#### API Versioning and Backwards Compatibility
Whenever an API is updated, a new version of the specification and the documentation is published. All API specifications are versioned. The guiding principle for the evolution of the API is that all updates are made to avoid “breaking changes”, i.e., changes that cause problems for clients that were developed according to previous versions. Thus, new functionality is mainly exposed as new resources or new attributes/fields in existing data structures. Only if imperative, to ensure the security or future maintainability, breaking API changes will be introduced. Because of this principle, users of the API are required to ignore fields in data structures that they do not recognize, unless otherwise noted in the documentation. Furthermore, the users of the API are to rely only on documented behavior, and to ignore absence of resources or functionality that is not documented. The documentation will state documented behavior as requirements.
In the documentation APIs are versioned as a semantic versioning scheme (“major.minor.revision”). Breaking changes are signaled by increasing the major version number. This is expected to be a rare occurrence after the development phase is over. If only the minor or the revision number is updated, existing clients targeting the major version number will be compatible with the new version of the API. The URLs exposed by the API contain, as their first sub-resource, a version number which reflects the major version. This is initially “v1”, reflecting the first version of the API. If no breaking changes are introduced, this number will stay at “v1.

#### OAuth 2.0 Authorization Framework
All APIs are protected using the OAuth 2.0 authorization framework.
The Token endpoint is the entry point for getting access- or service tokens issued, which is then used as authorization bearer tokens.
Access tokens are retrieved from end-user authentication flows or via the “Client Credentials Grant” type flows at the Token endpoint. Service tokens are always retrieved from the Token endpoint using the “Client Credentials Grant” type flow.

### Broker API – OpenID Connect endpoints
In this section the available OpenID Connect endpoints will be listed.
All listed endpoints in this section will conform to the OpenID Connect specification and will be listed in the Discovery endpoint.

#### Discovery endpoint
The “OpenID Connect Discovery” endpoint. See [Environments](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/environments.html) for reference.
Signaturgruppen Broker uses OpenID Connect Discovery which allows for automatic retrieval and dynamic changes of endpoints, cryptographic primitives, supported scopes and other features.
All Broker API endpoints have their endpoints published via the Discovery endpoint.
The Discovery endpoint will dynamically list available endpoints, available scopes and various other relevant OIDC specific information.

#### Token endpoint
All tokens issued by the Signaturgruppen Broker platform is issued from the Token endpoint. The endpoint follows the OpenID Connect specification.

### UserInfo endpoint
For most user authentication flows, the resulting access token provides access to the UserInfo endpoint by using the access token as an authentication bearer token.
The endpoint returns the full list of issued user-claims for the end-user, if the end-user session is active at Signaturgruppen Broker.
The result from the Userinfo endpoint may contain the following information
User-claims from the associated end-user session.
Session info from the associated end-user session.
Transaction specific claims, like signing texts – bound to the specific access token.
A result from the User Info endpoint after a successful MitID authentication flow with scope=”openid mitid ssn” could look like this:

```
{
  "sub" : "bab646bb-8608-4ac7-ac42-cee4ad490600",
  "mitid_uuid" : "af0196a3-6c61-464d-ab04-6394191a753d",
  "mitid.age" : "35",
  "mitid.date_of_birth"	: "1985-03-29",
  "mitid.ial_identity_assurance_level" : "SUBSTANTIAL”,
  "da.cpr" : "290385-xxxx",
  "mitid.identity_name" : "Hans Hansen",
  "session_status" : "active",
  "session_identifier" : "19712D37-0C76-4AAA-B049-BACD585F484F"
}
```

Specific Userinfo endpoint claims

| Claim | Value |
| --- | --- |
| idp_identity_id | The identity provider specific identifier for the end-user. This is typically the same as some other identity provider specific claim also issued (based on scopes requested). For example, the Danish MitID identity provider issues a global UUID as identifier for the MitID identity, which is set in the mitid.uuid claim for MitID flows, here the idp_identity_id also contains the mitid.uuid. This claim can be used as a generic identity provider specific identification of the end-user identity. |
