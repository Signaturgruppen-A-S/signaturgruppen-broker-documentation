
# CIBA + PAR integration
Signaturgruppen Broker supports a special [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) integration, that allows a service provider to initate any flow normally available via Signaturgruppen Brokers OpenID Connect interface via CIBA.

This allows a purely backend initiated flow in which a backend service is able to initiate a flow and continuously poll for flow completion and upon completion receive the normal Token ednpoint result (OIDC tokens).
Signaturgruppen Broker has implemented this by introducing a special API endpoint that allows a service to convert an initiated CIBA flow into a PAR authentication URL which can be opened in any supported browser in which the end-user will complete the requested flow.

This can help various setups to integrate flows into a range of integrations with better compatability across applications such as mobile apps, web applications and backend services. This also allows for setting up flows where the end-user completes the flow on a different device and/or browser than from where the flow is initiated, allowing for better flexibility and support for various flow types. 

# CIBA integration
Signaturgruppen Broker supports CIBA integration for various flows and follows the [CIBA spec](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) with the CIBA endpoint reference found via the discovery endpoint, see environment section for reference to the [authority URL](https://signaturgruppen-a-s.github.io/signaturgruppen-broker-documentation/environments.html) of Signaturgruppen Broker.

## CIBA parameters
The **scope**, **client_..** and **grant_type** parameters is set as a top-level CIBA request parameters, with the same effect as for the corresponding OIDC flow.
The **login_hint_token** parameter is required for the CIBA initialization invocation. 
Any other top-level parameters normally set for Signaturgruppen Broker OIDC integrations is set as part of the **login_hint_token** object.

### **login_hint_token** structure
Following a non-exhaustive table of options:
| Parameter | Values | Required | Description |
|----------|----------|----------|----------|
| flow_type    | broker_oidc   | Required   | Must be set to the value **broker_oidc**   |
| idp_values    | Single space-delimited string | Optional   | Example: "mitid" or "mitid mitid_erhverv"   |

The **login_hint_token** must be constructed as a valid JSON structure, setting each of the required and optional values as JSON key-value pairs.
For example: 
```
login_hint_token={"flow_type": "broker_oidc" }
```

## Example MitID flow
The following examples demonstrate how to setup a MitID login flow using the CIBA PAR mechanics, setting up appropriate values for the flow.

- Initiate CIBA flow, retrieve **request_id**
- Initiate OIDC PAR flow using **request_id**
- Poll result examples (pending, error, success)

### Initiate CIBA flow, retrieve **request_id**:

```
curl --location 'https://pp.netseidbroker.dk/op/connect/ciba' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=urn:openid:params:grant-type:ciba' \
--data-urlencode 'scope=openid mitid' \
--data-urlencode 'client_id=b....7' \
--data-urlencode 'client_secret=a....SuaQ==' \
--data-urlencode 'login_hint_token={"flow_type": "broker_oidc" }' \
--data-urlencode 'prompt=login' \
--data-urlencode 'idp_values=mitid'
```
