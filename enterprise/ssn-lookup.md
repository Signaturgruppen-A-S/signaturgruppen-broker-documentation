---
title: SSN Lookup
layout: home
parent: Enterprise features
nav_order: 6
---

# SSN Lookup
As outlined in [SSN details claims for MitID flows](../idps/mitid.html#ssn-details-claims-for-mitid-flows) it is possible to request detailed information about a person's name and address by including the OpenID Connect scopes `ssn.details_name` and `ssn.details_address` in the authentication request.
The Signaturgruppen Broker uses the GCTP service offered by CPR as source for these details.

## Test data
Both test environments of Signaturgruppen Broker (FT & PP) are integrated with CPR's Demo environment.
The available test data from CPR is documented at https://cprservicedesk.atlassian.net/wiki/spaces/CPR/pages/11436127/Testdata

If no data is available in CPR based on the CPR returned by MitID the `ssn.details.status`-claim will contain the value `unable_to_lookup`.

It is possible to query if an identity exists in the MitID Test Tool using a CPR/SSN, in the same manner with which you query based on identity claim or MitID UUID.
It is not possible to create multiple MitID Test Tool identities with the same SSN/CPR.

> Please note that there for test environments are not coordination of data provided by MitID and CPR. Hence, full name from MitID may not match the components provided in the ssn.details-claims, which originate from CPR.

> Please note that all test data is fictional and does not contain personal data.

## Operational status
Signaturgruppen does not provide operational status for CPR, but encourage our customers who rely on this data in their authentications to subscribe CPR's newsletters and status pages:

- Newsletter: https://www.cpr.dk/cpr-nyt/nyhedsbreve
- Status page (PP): https://cprdkdemo.statuspage.io/
- Status page (Production): https://cprdkprod.statuspage.io/

Should you encounter problems which has not been announced by a newsletter or on the status page you are of course welcome to contact Signaturgruppen.