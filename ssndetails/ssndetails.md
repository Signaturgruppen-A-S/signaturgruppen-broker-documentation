---
title: SSN Details
layout: home
parent: API integration
nav_order: 1
---

# SSN Details API Documentation

This API provides address information based on Danish CPR (Central Person Registry) numbers.

> Swagger definition (for PP environment) found at https://pp.netseidbroker.dk/op/swagger/index.html

**Note**: Access requires service tokens (client credentials).

## Limitations

- **Consent Required**: You must have consent from the CPR number holder to use this API.
- **Invalid CPR Requests**: Making more than 5 requests with unknown CPR numbers within 24 hours will block your access for 24 hours.
- **Support**: If you are blocked accidentally, contact [support@signaturgruppen.dk](mailto:support@signaturgruppen.dk).

## Usage

### 1. Calling the API

Use your service token to access the protected resource.

#### Example Request

```http
POST /api/v1/ssn-details HTTP/1.1
Host: https://pp.netseidbroker.dk/op/
Authorization: Bearer {Your_Service_Token}

{
    "Cpr": "1234567890"
}
```

#### Example Response

```json
{
    "ssn.details.status": "success",
    "ssn.details.name_address_protected": "false",
    "ssn.details.person_status": "Active",
    "ssn.details.given_name": "John",
    "ssn.details.surname": "Doe",
    "ssn.details.care_of_name": null,
    "ssn.details.city_name": "Copenhagen",
    "ssn.details.street_name": "Main Street",
    "ssn.details.zip_code": "1234",
    "ssn.details.zip_name": "Copenhagen"
}
```

_Note: Actual values have been replaced with placeholders._

### 2. Calling the API with an Invalid CPR Number

#### Example Request

```http
POST /api/v1/ssn-details HTTP/1.1
Host: https://pp.netseidbroker.dk/op/
Authorization: Bearer {Your_Service_Token}

{
    "Cpr": "0101019999"
}
```

#### Example Response

```json
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
    "errorMessage": "Failed to lookup CPR. You have a maximum of 5 failed CPR requests within 24 hours."
}
```

### 3. Exceeding Invalid CPR Request Limit

#### Example Request

```http
POST /api/v1/ssn-details HTTP/1.1
Host: https://pp.netseidbroker.dk/op/
Authorization: Bearer {Your_Service_Token}

{
    "Cpr": "0101019999"
}
```

#### Example Response

```json
HTTP/1.1 429 Too Many Requests
Content-Type: application/json

{
    "errorMessage": "Too many requests with invalid CPR numbers. Expired until: 2024-11-15T13:14:08.2277904+00:00"
}
```

## Response Details

| Field                                 | Description                                                      |
|---------------------------------------|------------------------------------------------------------------|
| `ssn.details.status`                  | `success` or `unable_to_lookup`                                  |
| `ssn.details.name_address_protected`  | `true` or `false`                                                |
| `ssn.details.person_status`           | See [Person Status Values](#person-status-values)                |
| `ssn.details.given_name`              | The person's given (first) name                                  |
| `ssn.details.middle_name`             | The person's middle name                                         |
| `ssn.details.surname`                 | The person's surname (last name)                                 |
| `ssn.details.care_of_name`            | Care-of name (c/o)                                               |
| `ssn.details.city_name`               | City name                                                        |
| `ssn.details.floor`                   | Floor number                                                     |
| `ssn.details.street_name`             | Street name                                                      |
| `ssn.details.street_number`           | Street number                                                    |
| `ssn.details.unit`                    | Unit or apartment number                                         |
| `ssn.details.zip_code`                | Postal code                                                      |
| `ssn.details.zip_name`                | Name corresponding to the postal code                            |

**Note**: Only fields with values are returned.

### Special Cases

- **Name and Address Protected**: If `ssn.details.name_address_protected` is `true`, only the following fields are returned:
  - `ssn.details.status`
  - `ssn.details.name_address_protected`
  - `ssn.details.person_status`

- **Emigrated Individuals**: If `ssn.details.person_status` is `'Emigrated'`, only these fields are returned:
  - `ssn.details.status`
  - `ssn.details.name_address_protected`
  - `ssn.details.person_status`
  - `ssn.details.given_name`
  - `ssn.details.middle_name`
  - `ssn.details.surname`

### Person Status Values

The `ssn.details.person_status` field can have the following values:

- `Active`
- `ActiveWithHighRoadCode`
- `ActiveRegisteredResidenceInGreenlandicCpr`
- `ActiveRegisteredResidenceInGreenlandicCprWithHighRoadCode`
- `ActiveAdministrativeSocialSecurityNumber`
- `CancelledSocialSecurityNumber`
- `DeletedSocialSecurityNumber`
- `ChangedSocialSecurityNumber`
- `Missing`
- `Emigrated`
- `Deceased`

## Contact Information

For assistance, please contact [support@signaturgruppen.dk](mailto:support@signaturgruppen.dk).
