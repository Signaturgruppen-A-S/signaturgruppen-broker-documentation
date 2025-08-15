
# CIBA + PAR integration
Signaturgruppen Broker supports a special [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html) integration, that allows a service provider to initate any flow normally available via Signaturgruppen Brokers OpenID Connect interface via CIBA.

This allows a purely backend initiated flow in which a backend service is able to initiate a flow and continuously poll for flow completion and upon completion receive the normal Token ednpoint result (OIDC tokens).
Signaturgruppen Broker has implemented this by introducing a special API endpoint that allows a service to convert an initiated CIBA flow into a PAR authentication URL which can be opened in any supported browser in which the end-user will complete the requested flow.

This can help various setups to integrate flows into a range of integrations with better compatability across applications such as mobile apps, web applications and backend services. This also allows for setting up flows where the end-user completes the flow on a different device and/or browser than from where the flow is initiated, allowing for better flexibility and support for various flow types. 

## CIBA integration
