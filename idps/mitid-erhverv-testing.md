---
title: MitID Erhverv Testing
layout: home
parent: MitID Erhverv
grand_parent: Identity providers
nav_order: 1
---

# MitID Erhverv Testing and Test-Users**

## Introduction

This document describes the various options for MitID Erhverv. Note that the document is only intended to introduce the MitID Erhverv setup as well as how it works in combination with the Nets eID Broker.

The intended audiences are IT developers, architects, and integration testers.

Business functionality specified in this document may be subject to different commercial agreement requirements.

General information, online demonstration, documentation (including the newest version of this document) and example code is found at <https://broker.signaturgruppen.dk>.

Note that for in-depth information on Nemlog-in and the MitID Erhverv solution the reader is referred to <https://digst.dk/it-loesninger/nemlog-in/>.

## MitID Erhverv and NemLog-In EIA

As mentioned, it is expected that the reader will have some knowledge of how NemID, MitID and NemLog-In work. This section serves as a summary. Private citizens can use MitID authentication to login with their private identity. Employees in companies can login in different ways with MitID than was the case with the previous NemID authentication setup, where only dedicated NemID OTP/keyfile could be used. The different options are listed below. Regardless of how the user logs in to his MitID Erhverv role the service provider will get access to the same identity information of the employee.

### MitID Private ID authenticators

If both the user and the company where the user is employed agree to use this option, it is possible to let the user re-use his private identity and corresponding MitID authenticators to login.

Note also that private MitID or NemID logins can be stepped up through the so-called private-to-business mechanism where companies associated to the users CPR number can be looked up in the Erhvervsstyrelsen company register. This is a different mechanism than MitID Erhverv and will require the user to enter his CPR number. It is expected that these company associations will be a part of the standard MitID Erhverv at some point.

### MitID Erhverv dedicated authenticators

The company administrator can assign a dedicated MitID Authenticator to the user. This could be any of the approved MitID authenticators known from the MitID Private solution. This can be:

1. MitID Authenticator App
2. MitID OTP device/code reader
3. MitID Chip

The login process is similar to that of ordinary MitID known for private IDs.

### NSIS Local IdP

Some large user organizations are expected to implement a so-called identity provider (IdP). Read more on the solution at: <https://digst.dk/it-loesninger/nemlog-in/om-loesningen/aendring-i-funktionaliteter/lokal-idp/> With a local IdP the user organization has the responsibility to verify the user identity and provide the user with authenticators to be able to authenticate the user in a secure manner to obtain the various security levels identified in the NSIS standard framework. With a local IdP the login process is handled between Nemlog-in and the local IdP, however the service provider will after successful authentication receive the usual set of user information.

## User administration in NemLog-In EIA

Employees are administered in the EIA-system by company administrators. This process is not described in this document. Note that an employee can be assigned any combination of the above-mentioned authentication setups. For testing MitID Erhverv users can log-in to their self-service at:

<https://erhvervsadministration.test-devtest4-nemlog-in.dk/>

## MitID Erhverv integration in Nets eID Broker

For a service provider to be able to use MitID Erhverv identities through the Nets eID Broker a special scope (nemlogin) should be used to get the relevant employee information and further the login process is initiated using a special identity provider through the configuration of idp_value (mitid_erhverv). The login process will then support the above-mentioned ways of logging in, however currently the local IdP option is not available in any of the Nemlog-in systems. Note that MitID Erhverv will need to be set up for a given service provider by the Nets eID Broker administration before it can be used.

When the user is logging in, he will be prompted to select between identities assigned to the used authenticator if more than one is available. Note that if a MitID Private authenticator is used the integration can be setup to allow the user to not complete the anticipated MitID Erhverv login but instead complete the login as the associated private identity.

### Returned claims

If the user ends up logging in to his employee identity the following

special claims are returned from the user info endpoint:

| **Claim** | **Value** |
| --- | --- |
| nemlogin.date_of_birth | Employee birthday |
| nemlogin.email | Employee e-mail (may be empty) |
| nemlogin.name | Employee Name (note for anonymous users) |
| nemlogin.family_name | Employee Family Name (note for anonymous users) |
| nemlogin.given_name | Employee First Name (note for anonymous users) |
| nemlogin.nemid.rid | Employee certificate RID from NemID migration (or assigned) |
| nemlogin.org_name | Company Name |
| nemlogin.persistent_professional_id | MitID Erhvervs Global UUID/ID from EIA |
| nemlogin.cvr | Company CVR |
| nemlogin.se_number | Company SE number (may be empty) |
| nemlogin.p_number | Company P number (may be empty) |

To be able to recognize the user on sub_sequent logins it is recommended to persist the persistent_professional_id which is the global identifier. However, as with other logins through the Nets eID Broker the sub-claim from the id_token can also be used to recognize the user. Note also that it is also recommended to verify that the expected identity type is returned as a result of the login process.

## MitID Erhverv test-users

The Nets eID Broker test/pp-system is wired up to use the so-called NemLog-In test-devtest4 environment. This relies on a couple of sub-systems. However, to test basic functionality it is not important to know anything else than how to use the ordinary MitID PP test-tool. For testing, some test identities have been set up and details are provided in the tables below. To be able to test migration of existing users NemID Erhverv users a few of the users are equipped with corresponding NemID Erhverv key-files which can be located on the provided links.

**User with both MitID Private and dedicated authenticator**

1. CPR: 0502669796  
    MitID Privat ID: Abenaa133  
    MitID Erhverv ID: Abenaa133erhverv  
    MitID Erhverv EIA UUID: 7b10ffa7-d26f-40a2-b7a9-ae3f3231cdcc (CVR: 43950932)  
    NemID Erhverv RID: 93068542 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Abenaa_Petersen_43950932.zip>" (PW: Test1234))
2. CPR: 0907549089  
    MitID Privat ID: Conrad79491  
    MitID Erhverv ID: Conrad79491erhverv  
    MitID Erhverv EIA UUID: acd3e77b-ec38-4715-8804-6c11429faabe (CVR: 43950932)  
    NemID Erhverv RID: 19766788 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Conrad_Petersen_43950932.zip>" (PW: Test1234))
3. CPR: 1705969945  
    MitID Privat ID: Kaj668  
    MitID Erhverv ID: Kaj668erhverv  
    MitID Erhverv EIA UUID: 0fa1da81-c623-47d9-a274-e20c71f851de (CVR: 43950932)  
    NemID Erhverv RID: 79095121 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Kaj_Andersen_43950932.zip> (PW: Test1234))

**User with only MitID Private authenticator**

1. CPR: 2707343888  
    MitID Privat ID: Freja5669  
    MitID Erhverv EIA UUID: 337e05cb-e2e5-4dea-bd6c-57b788bcd841 (CVR: 43950932)  
    NemID Erhverv RID: 82864416 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Freja_Henriksen_43950932.zip> (PW: Test1234))
2. CPR: 2707283583  
    MitID Privat ID: Stig9431  
    MitID Erhverv EIA UUID: b557c19c-ca0a-4128-ae3f-c81b76c1290f (CVR: 43950932)  
    NemID Erhverv RID: 21309740 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Stig_Henriksen_43950932.zip> (PW: Test1234))
3. CPR: 1007899315  
    MitID Privat ID: Earl112066  
    MitID Erhverv EIA UUID: f1004927-e6dc-41a3-af6a-8b31d27ae99f (CVR: 43950932)  
    NemID Erhverv RID: 30663772 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Earl_Nielsen_43950932.zip> (PW: Test1234))
4. CPR: 2103602680  
    MitID Privat ID: Buggi2831  
    MitID Erhverv EIA UUID: 338730b4-fcca-499f-ae7f-c67187edc3ce (CVR: 43950932)  
    NemID Erhverv RID: 69271471 (CVR: 43950932) ([https://www.signaturgruppen.dk/download/mitiderhverv/ Buggi_Poulsen_43950932.zip](https://www.signaturgruppen.dk/download/mitiderhverv/%20Buggi_Poulsen_43950932.zip) (PW: Test1234))
5. CPR: 0801105610  
    MitID Privat ID: Carinalouissa6583  
    MitID Erhverv EIA UUID: 96436aa2-6d6f-410b-8856-26a209eaed1f (CVR: 43950932)  
    NemID Erhverv RID: 71459852 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Carinalouissa_Christensen_43950932.zip> (PW: Test1234))
6. CPR: 2008933068  
    MitID Privat ID: Sofia7798  
    MitID Erhverv EIA UUID: 2336f5e6-bf21-45ad-ac7d-887168cd321e (CVR: 43950932)  
    NemID Erhverv RID: 12693636 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Sofia_Poulsen_43950932.zip> (PW: Test1234))
7. CPR: 2708453261  
    MitID Privat ID: Sofus47825  
    MitID Erhverv EIA UUID: 3fb1cd3c-45ce-40d2-ba5b-665a8dcb08f2 (CVR: 43950932)  
    NemID Erhverv RID: 46121612 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Sofus_Hansen_43950932.zip> (PW: Test1234))
8. CPR: 1004240916  
    MitID Privat ID: Clara9572  
    MitID Erhverv EIA UUID: 0e5ffdf0-64ab-45a1-adc0-88b09687703e (CVR: 43950932)  
    NemID Erhverv RID: 83088158 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Clara_Joergensen_43950932.zip> (PW: Test1234))
9. CPR: 2212923373  
    MitID Privat ID: Anders4329  
    MitID Erhverv EIA UUID: bd0f36f2-1ab7-43ee-afd5-ac2fd79f84f8 (CVR: 43950932)  
    NemID Erhverv RID: 54591620 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Anders_Henriksen_43950932.zip> (PW: Test1234))

**User with only dedicated MitID employee authenticator**

1. CPR: 1806924233  
    MitID Erhverv ID: Benny9430erhverv  
    MitID Erhverv EIA UUID: 58a162b4-8a0e-4254-9d32-4ecf6adb8562 (CVR: 43950932)  
    NemID Erhverv RID: 99610558 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Benny_Poulsen_43950932.zip> (PW: Test1234))
2. CPR: 2911310158  
    MitID Erhverv ID: Olivia282erhverv  
    MitID Erhverv EIA UUID: 1341af14-041c-414f-8bf0-b617f270f1fb (CVR: 43950932)  
    NemID Erhverv RID: 77310023 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Olivia_Andersen_43950932.zip> (PW: Test1234))

**Users with multiple company identities**

1. CPR: 1902078084  
    MitID Privat ID: Laura40305  
    MitID Erhverv EIA UUID: 8d3110d7-a2ad-423d-8590-67f22939178e (CVR: 43950932)  
    NemID Erhverv RID: 40854143 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Laura_Nielsen_43950932.zip> (PW: Test1234))

MitID Erhverv EIA UUID: 2a4896a6-f69c-445e-858c-33eefee9c947 (CVR: 16662879)

NemID Erhverv RID: 99612445 (CVR: 16662879)

1. CPR: 1405509683  
    MitID Privat ID: Bjørn2712  
    MitID Erhverv EIA UUID: 0c23111c-928b-4adb-a413-e07006c48c16 (CVR: 43950932)  
    NemID Erhverv RID: 10438898 (CVR: 43950932) (<https://www.signaturgruppen.dk/download/mitiderhverv/Bjoern_Jensen_43950932.zip> (PW: Test1234))

MitID Erhverv EIA UUID: 20ad8d25-dbe7-4244-ba72-f1d2f6aa886f (CVR: 16662879)

NemID Erhverv RID: 81440213 (CVR: 16662879)

## Specific configurations

Below is a list of relevant configurations to consider when planning your employee login solution. For reference a few configurations for NemID Erhverv are also described.

**Basic MitID Erhverv configuration**  
In order to receive only basic MitID Erhverv logged-in users the configuration below can be used. Note that it will only allow users with a valid MitID Erhverv Identity to log in (based on either a private MitID authenticator or a dedicated MitID Erhverv authenticator.

idp_values: mitid_erhverv  
scope: openid nemlogin

**MitID Erhverv configuration allowing also private MitID users**  
Besides the basic MitID Erhverv users private MitID users will also be allowed. Note that the identity information is received in mitid.-claims. Private MitID sessions can be stepped up with the private_to_business-scope.

idp_values: mitid_erhverv  
scope: openid mitid nemlogin  
idp_params: “mitid_erhverv”:{“allow_private”:true}

**NemID Erhverv configuration allowing also private NemID users**  
Besides the basic NemID Erhverv users private NemID users with mappings to companies will also be allowed. Private NemID sessions will be attempted to be stepped up with the private_to_business-scope. Note that a user that logs in with a private NemID identity in order to attempt private_to_business step-up may continue without any such mapping.

idp_values: nemid  
scope: openid nemid private_to_business  
idp_params: "nemid":{"amr_values":"nemid.otp nemid.keyfile"}  

**Basic NemID configuration  
**Besides the basic NemID Erhverv users private NemID users will also be allowed. For private identities the SSN/CPR-step-up process can be followed.

idp_values: nemid  
scope: openid nemid ssn  
idp_params: "nemid":{"amr_values":"nemid.otp nemid.keyfile"}
