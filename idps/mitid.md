---
title: MitID
layout: home
parent: Identity providers
has_children: true
nav_order: 1
---

## MitID

The MitID identity provider is the official Danish national electronic identity, replacing NemID.

More information is found here: <https://digst.dk/it-loesninger/mitid/>.

MitID follows the “National Standarder for Identiteters Sikringsniveauer” (NSIS) and all MitID flows is mapped to one of authentication Level of Assurance’s (LoA) found in the NSIS specification: <https://digst.dk/it-loesninger/nemlog-in/det-kommende-nemlog-in/vejledninger-og-standarder/nsis-standarden/>.

### MitID official communication material

A zip file containing the official communication kit for MitID can be [downloaded here](https://signaturgruppen.dk/download/broker/docs/mitid-kommunikationsmaterialer-v01.zip)

### Security Advice for MitID Users

The security advice below will help protect the MitID user against frauad both during the migration proces and in the subsequent use. 

Below you will find material from MitID, which you can use when communicating with your users with respect to MitID through relevant channels.

The "Look out for MitID" animation:

There are two animations tht give the user valuable information as of how to protect their digital lives including MitID. The videos are in Danish:

Long version at MitID.dk: <https://www.youtube.com/watch?v=4HN3N3xJ9W8>
Short versiont at MitID Youtube channel: <https://www.youtube.com/watch?v=8pu8LefqVlY>

### Supported browsers

MitID only test and support the browsers that have at least 2% market share and thus it is a requirement, that app switch integrations utilize the default browsers of the respective OS in question.

It is a security requirement that the end-user is presented with the address bar from the browser to allow the end-user to verify the <https://mitid.dk> domain during the MitID flow.

Further it is a requirement that the browser utilized supports all the MitID authenticator options, including the MitID chip which is based on Web Authentication.

Embedded browsers are not supported and not allowed (\* they are allowed in iframe flow variants) – thus integrations must adhere to the Android Custom Tabs or iOS SFSafariViewController browser integrations and it not allowed to use a standard embedded browser for the integration to MitID.

### Supported OIDC parameters

| **Request parameter** | **Description** |
| --- | --- |
| idp_values | **mitid** |

### Supported identity provider parameters (idp_params -> mitid)

<table><tbody><tr><th><p><strong>Identity Provider parameters (mitid)</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>reference_text</p><p>Note: <strong>This text will be displayed to the user in all MitID flows inside the MitID client, but only at the last authenticator shown.</strong></p></td><td><p>Type: Base64 encoded string.</p><p>The reference text containing the transaction content (e.g., “Transfer &lt;amount&gt; to &lt;ac-count&gt;”).</p><p>It will be shown to the user in the MitID App.</p><p>It is limited to 130 characters.</p></td></tr><tr><td><p>transaction_text</p><p><em>This parameter will be ignored for unsigned requests.</em></p></td><td><p>Type: Base64 encoded string.</p><p>The transaction text is presented to the end-user as part of the MitID flow and allows service providers to provide a transactional context for the MitID flow.</p><p>This can be of the types valid for the transasction_text_type parameter.</p></td></tr><tr><td><p>transaction_text_type</p></td><td><p>Type: String</p><p>Specifies the type of the transaction_text parameter.</p><p>One of</p><ul><li>text</li><li>html</li></ul><p>If text is specified, the transaction_text will be displayed “as-is” without any rendering, as a plain text value.</p><p>If html is specified, the content will be displayed and rendered as HTML. The allowed HTML is restricted as specified in this document.</p></td></tr><tr><td><p>uuid_hint</p></td><td><p>Type: String</p><p>If this parameter is set with the end-user MitID UUID, the MitID flow will be automatically started for the MitID identity with the specified MitID UUID.</p></td></tr><tr><td><p>cpr_hint</p><p><em>It is required encrypt requests when sending cpr_hint, see </em><strong><em>[NEB-TECHREF]</em></strong><em> for reference.</em></p></td><td><p>Type: String</p><p>If this parameter is set with the end-user CPR, the MitID flow will be automatically started for the MitID identity with the specified CPR.</p></td></tr><tr><td><p>require_psd2</p></td><td><p>Type: Boolean</p><p>If this parameter is set to true, the individual authentication flow will be PSD2 compliant by restricting what information can be revealed to the end-user during authentication.</p></td></tr><tr><td><p>loa_value</p></td><td><p>Specifies the requested Level of Assurance level for the MitID flow.</p><p>One of</p><ul><li>low</li><li>substantial</li><li>high</li></ul><p>if both loa_value and aal_value is undefined in the request, the default will be set to loa_value=substantial.</p></td></tr><tr><td><p>aal_value</p></td><td><p>Specifies the requested Authenticator Assurance Level for the MitID flow.</p><p>One of</p><ul><li>low</li><li>substantial</li><li>high</li></ul><p>if loa_value is set, aal_value will be ignored.</p><p>This enables to request a MitID flow where the aal level might be higher than the (expected) Level of Assurance (loa), i.e. ial=low, aal=substantial =&gt; loa=low.</p></td></tr><tr><td><p>enable_step_up</p><p><em>Requires prompt=login</em></p><p><em>Requires loa_value or aal_value higher than the existing session from which to step-up from.</em></p></td><td><p>Type: bool</p><p>Specifies that the MitID step-up functionality is requested.</p><p>To activate the MitID step-up flow the prompt=login must be specified to instruct the flow to reauthenticate. Then if the enable_step_up is set to true and if the requested loa og aal value is higher than the existing session for the end-user the MitID authentication will start in step-up mode.</p></td></tr><tr><td><p>action_text</p></td><td><p>Type: string, default: null</p><p>Allows to specify one of the accepted “action” header texts for the MitID client flow.</p><p>Must be one of the following</p><ul><li>"LOG_ON"</li><li>"APPROVE"</li><li>"CONFIRM"</li><li>"ACCEPT"</li><li>"SIGN"</li></ul></td></tr></tbody></table>

### Example JSON for identity providers

```json
{
  "mitid": {
    "loa_value": "substantial",
    "enable_step_up": true,
    "uuid_hint": "efc7ffb4-e086-4f5f-a1d5-b3c7227db629"
  }
}
```

```
idp_params=%7B%E2%80%9Cmitid%E2%80%9D%3A%7B%E2%80%9Cloa_value%E2%80%9D%3A%E2%80%9Dsubstantial%E2%80%9D%2C%20%E2%80%9Cenable_step_up%E2%80%9D%3Atrue%2C%20%E2%80%9Cuuid_hint%E2%80%9D%3A%20%E2%80%9Cefc7ffb4-e086-4f5f-a1d5-b3c7227db629%E2%80%9D%7D%7D
```

### Supported scope values

<table><tbody><tr><th><p><strong>Scope</strong></p></th><th><p><strong>Description</strong></p></th></tr><tr><td><p>mitid</p></td><td><p>List of claims:</p><ul><li>mitid.uuid</li><li>mitid.date_of_birth</li><li>mitid.age</li><li>mitid.identity_name</li><li>mitid.ial_identity_assurance_level</li><li>mitid.transaction_id</li></ul></td></tr><tr><td><p>transaction_token</p></td><td><p>The transaction_token scope requests the transaction token from the Token endpoint including the following claims.</p><p>These claims are not shared in a SSO with other services.</p><ol><li>transaction_id</li><li>mitid.transaction_text</li><li>mitid.transaction_text_type</li><li>mitid.reference_text</li></ol></td></tr><tr><td><p>ssn</p></td><td><p>Social Security Number.</p><p>List of claims:</p><ul><li>dk.cpr</li></ul><p>Will trigger MitID CPR user-interaction.</p></td></tr></tbody></table>

### ID Token identity claims

<table><tbody><tr><th><p><strong>Claim value</strong></p></th><th><p><strong>Possible values</strong></p></th></tr><tr><td><p>identity_type</p></td><td><p>private</p></td></tr><tr><td><p>idp</p></td><td><p><strong>mitid</strong></p></td></tr><tr><td><p>loa</p></td><td><p>Level of Assurance</p><p>One of</p><ul><li>https://data.gov.dk/concept/core/nsis/Low</li><li>https://data.gov.dk/concept/core/nsis/Substantial</li><li>https://data.gov.dk/concept/core/nsis/High</li></ul></td></tr><tr><td><p>ial</p></td><td><p>Identity Assurance Level</p><p>One of</p><ul><li>https://data.gov.dk/concept/core/nsis/Low</li><li>https://data.gov.dk/concept/core/nsis/Substantial</li><li>https://data.gov.dk/concept/core/nsis/High</li></ul></td></tr><tr><td><p>aal</p></td><td><p>Authenticator Assurance Level</p><p>One of</p><ul><li>https://data.gov.dk/concept/core/nsis/Low</li><li>https://data.gov.dk/concept/core/nsis/Substantial</li><li>https://data.gov.dk/concept/core/nsis/High</li></ul></td></tr><tr><td><p>amr</p></td><td><p>The list of authenticators used to achieve the resulting LoA.</p><p>Possible values are:</p><ul><li>password</li><li>code_token</li><li>code_reader</li><li>code_app</li><li>code_app_enchanced</li><li>u2f_token</li></ul></td></tr><tr><td><p>mitid.psd2</p></td><td><p>The mitid.psd2 claim is only issued as an ID Token identity claim if the authentication of an end-user is PSD2 compliant.</p></td></tr></tbody></table>

###

Transaction token MitID specific claims

<table><tbody><tr><th><p><strong>Claim value</strong></p></th><th><p><strong>Possible values</strong></p></th></tr><tr><td><p>mitid.uuid</p></td><td><p>Same value as for ID token.</p></td></tr><tr><td><p>mitid.reference_text</p></td><td><p>Passthrough of the MitID <strong>reference_text</strong> identity provider parameter.</p></td></tr><tr><td><p>mitid.transaction_text_sha256</p></td><td><p>Base64 encoded SHA256 digest of the MitID <strong>transactiontext</strong> identity provider parameter.</p></td></tr><tr><td><p>mitid.transaction_text_type</p></td><td><p>Passthrough of the MitID <strong>transactiontexttype</strong> identity provider parameter</p></td></tr><tr><td><p>mitid.psd2</p></td><td><p>The mitid.psd2 claim is always issued as a transaction token MitID specific claim.</p></td></tr><tr><td><p>transaction_actions</p></td><td><p>Type: string (single value) or JSON list</p><p>Only set, if one or more of the following transaction actions where performed:</p><ul><li>mitid.login (Login completed)</li><li>mitid.reuse_jwt (Automatic reuse of existing session)</li><li>mitid.sso_login (SSO login completed)</li><li>mitid.controlled_transfer (Controlled Transfer completed)</li><li>mitid.transaction_signing (Transaction signing)</li><li>mitid.cpr_match (CPR match completed)</li><li>mitid.cpr_lookup (Automatic CPR lookup completed)</li></ul></td></tr></tbody></table>

### MitiD Transaction signing

Nets eID Broker supports a transaction signing flow which enables the end-user to approve a transaction text based on text or HTML, as part of the MitID authentication. Transaction signing flow is limited to only signed requests.

This is done by setting the **transaction_text** and **transaction_text_type** MitID identity provider parameters.

The end-user will be shown the text/HTML and will have to approve the text to complete the transaction.

MitID natively supports the **reference_text** (130 characters) parameter which enables a limited size and format to present the end-user with detailed information about the transaction.

If the transaction token is requested, a NEB sealed record of the transaction is returned including all the relevant parameters used to complete the transaction.

If **transaction_text_type** is set to _html_, the HTML content of the **transaction_text** is restricted to a set of qualified tags and parsed to protect against possible malicious content and flow breakage.

Allowed HTML tags:

1. _html, body, head, style, title, div, p, ul, li, h1, h2, h3, h4, h5, h6, table, font, tr, th, td, i, u, b, center, a, q, small_

Disallowed expressions and attributes:

1. CSS expressions and embedded script links for _style_ tag.
2. CSS expression attributes.
3. Any on- attributes, such as _onload, onclick etc._
4. Script link attributes, such as _src, dynsrc, lowsrc, javascript:_ etc.

### MitID CPR flow

CPR is available from MitID flows if you are a public service provider. In this scenario, NEB will set **dk.cpr** in the result, if requested via the **ssn** scope.

If you are a private service provider, the user’s CPR will not be available from the MitID system. In this scenario, a CPR Match service is provided (see MitID CPR Match API), available for MitID Brokers making it possible to match an active MitID session and CPR and verify if the supplied CPR matches the authenticated MitID identity and thus making it possible to verify if a MitID identity has the given CPR.xxxxxx.

NEB implements this as a natural part of the MitID flow and will ask the user for CPR when the service provider requests CPR with the **ssn** scope.

**Note**_, that it is supported to request CPR via the CPR flow by reauthenticating a user with the additional_ **_ssn_** _scope. In this case, NEB_ _will reuse the active MitID session and ask the user for CPR (but will not ask for login), do the required CPR Match verification, and return the CPR to the service. This enables services to only ask for CPR using the CPR flow when needed for specific users._

**Note** that MitID only allows MitID Match for 15 minutes after the MitID session was issued.

### MitID CPR Match API

See the swagger API reference for details.

The Broker API supports a “MitID CPR Match API” that allows services to match a CPR with a MitID authentication from NEB.

In this way, services can ask the user for CPR and then call the API with the access token retrieved from NEB for the user authentication as authorization header.

This also allows services to verify that an already known CPR matches the MitID identity in question.

**Note** that MitID only allows MitID Match for 15 minutes after the MitID session was issued.

If “cprNumberMatch” returns false, it means that it is not possible to match the “cpr” input parameter with the CPR-number of the user. MitID restricts CPR-matching to a maximum of three tries per session, in which case the endpoint will return the following response:

To reset the CPR matching exceeded limitation, the client must prompt the user for reauthentication using MitID.

### MitID SSO

NEB implements and supports MitID SSO and allows integrating services to utilize MitID SSO for automatic sharing MitID sessions in a SSO defined and managed by participating MitID Brokers.

Any client, service or service provider can be member of up to one MitID SSO Group and if so, will automatically map all MitID sessions to this MitID SSO and automatically reuse an existing session if already active within this MitID SSO.

It is possible to setup MitID SSO groups with other MitID Brokers and other MitID Broker services and service providers.

NEB will handle the required end-user consent, which is required when automatically reusing an active MitID session.

Note, contact Signaturgruppen if you plan to use this feature.

#### MitID SSO logout

It is required by all clients who utilize a MitID SSO to inform their MitID broker of logout events from the end-user.

Logout is handled either by sending the end-user to the End session endpoint with the original issued ID token or by calling the Logout endpoint with the original issued ID token. See general section about logout in this document for more details.

MitID SSO requires that all logout events are handled using Back-channel endpoints and thus enabling the termination of MitID SSO sessions without the need of the end-user browser. MitID SSO groups are by this forced to be Back-channel SSO groups.

The only way participating service providers can receive logout events is by registering a valid Back-channel endpoint for their integration at NEB.

Participating service providers will be able to get the session status from the Userinfo endpoint.

### MitID Controlled Transfer

With MitID Controlled Transfer, a requesting service provider can request and retrieve a “MitID Controlled Transfer Token Exchange Code” (MitID CT Exchange Code) from their MitID Broker. Then, another service provider can exchange this to a MitID authentication from their respective MitID Broker.

Flow:

- Service provider A requests a MitID CT Exchange Code from Broker A and specifies a Transfer Token Text
- Service Provider A redirects end-user to Service Provider B with the MitID CT Exchange Code and Transfer Token Text
- Service Provider B uses Broker B and the implementation provided by Broker B to exchange the MitID CT Exchange Code token and the Transfer Token Text to a MitID authentication enabling the end-user login at Service Provider B.

The protocol for exchanging MitID CT Exchange Code between service providers are up to the two exchanging service providers to define.

The protocol for retrieving or exchanging MitID CT Exchange Code between service providers and MitID brokers are up to each broker to define.

The specification for retrieval and exchange towards the NEB interface is specified here. Note that it is up to each agreement with other service providers, how the MitID CT Exchange Code is exchanged – this is not specified nor handled by NEB.

**NOTE**: The requesting service provider has a mandatory requirement of getting the correct consent from the end-user, before sending the end-user to another service provider with a MitID CT Exchange Code. The official description of the requirement is:

When the user is performing a controlled transfer from service provider A to service provider B, it is the responsibility of service provider A to get a consent from the end user, regarding eID attributes which service provider A has requested on initial request, since these attributes will be available to service provider B.

#### Requesting a Controlled Transfer Token Exchange Code

A Controlled Transfer Token Exchange Code is retrieved by calling the MitID Controlled Transfer Token Exchange Code endpoint (see the swagger API reference for details).

| **Parameters** | **Description** |
| --- | --- |
| targetBrokerId | MitID Broker ID of receiving MitID broker |
| targetServiceProviderId | MitID Service Provider ID of receiving service provider |
| transferTokenText | Type: String<br><br>The calling service provider must specify a Transfer Token Text and hand this out to the receiving service provider. The Transfer Token Text is restricted by MitID to a maximum of 130 characters. Any length above 130 will result in an unsuccessful request. |

### Exchanging a Controlled Transfer Token Exchange Code to a MitID login

As a service provider integrating to NEB, it is possible to exchange a MitID CT Exchange Code received from another service provider for a MitID login.

The MitID CT Exchange Code is used by NEB in exchange of a MitID authentication token from the MitID APIs and as such enables NEB to automatically login an end-user in exchange for the MitID CT Exchange Code.

The MitID CT Exchange Code is passed as a parameter to the MitID identity provider in the NEB OIDC specification, alongside the received Transfer Token Text, and will be used to login the end-user based on the MitID session used to create the MitID CT Exchange Code in the first place.

### NemID PID claim for MitID flows

It is possible to request the NemID PID claim (nemid.pid) for MitID flows.

This is done by specifying the scope nemid.pid which will trigger the relevant end-user flow required to return the nemid.pid claim along with the MitID login.

<table><tbody><tr><th><p><strong>NemID PID scope</strong></p></th><th><p><strong>Claims</strong></p></th></tr><tr><td><p>nemid.pid</p></td><td><ul><li>nemid.pid</li><li>nemid.pid_status</li></ul><p>If set, the nemid.pid claim contains the NemID PID.</p><p>nemid.pid_status can have one of the following values</p><ul><li>success</li><li>unable_to_lookup</li></ul></td></tr></tbody></table>

To retrieve the PID from a MitID login Nets eID Broker will have to lookup PID from a NemLog-In3 supplied supporting service using the end-user Danish CPR number. The end-user will be guided through the needed steps automatically when the nemid.pid scope is specified.

It is recommended, that the nemid.pid scope is only specified when the returned MitID identity is unknown to the service in question, as the end-user will have to enter his Danish CPR number when this scope is specified.

The nemid.pid scope can be used as a reauthentication scope (see section about reauthentication) and thus NEB will automatically reuse an available user session to ensure that the end-user does not need to authenticate with MitID additional times.

Note, that the scopes “nemid.pid” and “ssn” can be used together or separately. If ssn is specified that dk.cpr claim will be issued to the service provider, but both nemid.pid and ssn scopes will trigger CPR matching for the end-user. It is thus possible to minimize the data handed back to the service provider by controlling it this way.

If the NemID PID could not be retrieved the nemid.pid_status claim will indicate the result.

### NemID PID API for MitID flows

_Endpoint: /mitid/nemidPidLookup (see the swagger API reference for details)_

Allows a service provider to lookup the NemID PID based on a successful MitID session for the end-user.

The endpoint allows for an optional CPR parameter, which is used to complete the lookup.

If the ssn flow was used for the authentication flow the CPR is known by NEB for the current session and thus CPR is not needed for the lookup.

If the ssn flow was not used, the service provider must provide the correct CPR for the end-user as part of the lookup.

### SSN details claims for MitID flows

It is possible to request detailed information about the person based of the Danish social security system for MitID flows. Service providers can request name and/or address details for the person. To request the details the scopes ssn.details_name and ssn.details_address will trigger the relevant end-user flow required to return the detailed information for the name and address respectively along with the MitID login.

<table><tbody><tr><th><p><strong>SSN details scope</strong></p></th><th><p><strong>Claims</strong></p></th></tr><tr><td><p>ssn.details_name<br>ssn.details_address</p></td><td><p>Shared claims:</p><ul><li>ssn.details.status</li><li>ssn.details.name_address_protected</li></ul><p>The ssn.details.status-claim can have one of the following to values:</p><ul><li>success</li><li>unable_to_lookup</li></ul><p>If the value of the ssn.details.status-claim is unable_to_lookup, then no other claims are returned.</p><p>The ssn.details.name_address_protected-claim contains a boolean value. If the claim value is “true” then no other claims from ssn.details-scopes are returned.</p><p>The shared claims are only returned once even if both ssn.details-scopes are requested.</p></td></tr><tr><td><p>ssn.details_name</p></td><td><p>List of claims:</p><ul><li>ssn.details.given_name</li><li>ssn.details.middle_name</li><li>ssn.details.surname</li></ul></td></tr><tr><td><p>ssn.details_address</p></td><td><p>List of claims:</p><ul><li>ssn.details.care_of_name</li><li>ssn.details.city_name</li><li>ssn.details.floor</li><li>ssn.details.street_name</li><li>ssn.details.street_number</li><li>ssn.details.unit</li><li>ssn.details.zip_code</li><li>ssn.details.zip_name</li></ul></td></tr></tbody></table>

Please note that the transaction token will contain a transaction_action-claim with the value “ssn.details_lookup”, when one or both of the ssn.details-scopes are requested as outlined in the section “Transaction token MitID specific claims”.

To retrieve the detailed information for the person from a MitID login Nets eID Broker will have to look up the information from the Danish national social security system using the Danish CPR number. The end-user will be guided through the needed steps automatically when one of or both the ssn.details-scopes are specified.

It is recommended, that the ssn.details-scopes only are specified, when the returned MitID identity is unknown to the service in question, as the end-user will have to enter his Danish CPR number when these scopes are specified.

Note, that the scopes “ssn.details_name”, “ssn.details_address” and “ssn” can be used together or separately. If ssn is specified that dk.cpr claim will be issued to the service provider, but all scopes will trigger CPR matching for the end-user. It is thus possible to minimize the data handed back to the service provider by controlling it this way.

### MitID service provider parameters

| **Identity Provider parameters (mitid)** | **Description** |
| --- | --- |
| transfer_token_exchange_code | Type: string<br><br>The issued MitID Controlled Transfer Token Exchange Code<br><br>Example:<br><br>”38e195d6-14ad-4ed9-9f0b-d72a26c8ed95” |
| transfer_token_text | Type: string (non-empty)<br><br>A Controlled Transfer Token Text, which is up to the calling Service Provider to define and set.<br><br>This text must be handed out to the receiving service provider together with the MitID Controlled Transfer Token Exchange Code. |

Note: See general section on identity provider parameters for reference on how to set the parameters.
