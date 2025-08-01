## Endorsement Web API Documentation

> **This is the official documentation for the Endorsement Web API.**
>
> **Important:** This API is strictly scoped to **endorsement-related operations only** and must not interact with endpoints outside the Endorsement API boundary (e.g., core Policy APIs).

## Docs

**Base URLs:** 
 - **Dev:** https://demo-web.bastionhmo.com/
 - **Prod:** https://live-web.bastionhmo.com/

## 1 Auth

**Auth Types:**
  -  0. No auth
    - No authorization token is required
    
  -  I. `Api-key (Basic)`: `api-key: "Base64Encode(username:password)"`
     - Vendor's credentials (username and password) will be shared during onboarding.
     - Example: `api-key: "YWRQxQA3dvcmtaW46UGFzc=="` - after the credentials have been encoded

  - II. `Api-key (Bearer)`: `api-key: {{auth_token}}`
    - The `auth_token` can be retrieved as described in session 1.1 below
    - Example: `api-key: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwHMiLCJpYXQiOjE2NjAwNTcm92aWRlciI6InVuaWZpZWRfcGF5bWVudA1MTk2MjN9.AS13xG_MOXTKKfpYKGYHfArjPULtq4uMJ_CNpa_ACBp"`


**Request Headers:**
  - `Content-Type`: application/json
  - `api-key`: AuthType I or II
 
 

### 1.1 `Generate auth token`

- **Auth:** `Auth Type I`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/auth/generateAuthToken`
- **Request Body:**
  - `expiresIn` (type: number, optional) - Time token expires in seconds. When no value is set, the token does not expire.

**Example Usage:**

```http
POST /api/v2/auth/generateAuthToken

{
  "expiresIn": 3600
}
```

**Example Response:**

```json
{
  "status": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### 1.2 `Change Password`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/auth/change-password`
- **Request Body**:
  - `oldPassword` (required) - New password for the vendor.
  - `newPassword` (required) - New password for the vendor.
  - `newConfirmPassword` (required) - New password for the vendor.

**Example Usage:**

```http
POST /api/v2/auth/change-password
{
  "oldPassword": "formerSecurePassword",
  "newPassword": "newSecurePassword",
  "newConfirmPassword": "newSecurePassword"
}
```

**Example Response:**

```json
{
  "data": {
    "success": true,
    "message": "Password Successfully updated!"
  }
}
```

---

### 1.3 `Reset password`

- **Auth:** `Auth Type 0 - No auth required`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/auth/password-reset?email={email}`
- **Query Parameters:**
  - `email` (required) - Email of the vendor.

**Example Usage:**

```http
GET /api/v2/auth/password-reset?email=johndoe@getnada.com
```

**Example Response:**

```json
{
  "status": true,
  "message": "Successful!"
}
```

---

## 2 Broker Policy

### 2.1 `Get broker plans`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/plans`

**Example Usage:**

```http
GET /api/v2/vendors/plans
```

**Example Response:**

```json
{
  "status": true,
  "message": "Broker Plans Retrieved Successfully!",
  "data": [
    {
      "id": 10052,
      "name": "Gold Plan",
      "description": "Lorem ipsum dolor sit amet...",
      "category": "string",
      "state": "Published",
      "premium": 64500.00
    },
  ]
}
```

---

### 2.2 `Get broker plan`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/plans/:id`
- **Parameters:**
  - `id` (required) - ID of the plan retrieved from the `GET broker plans` API, section 2.1 above.

**Example Usage:**

```http
GET /api/v2/vendors/plans/:id
```

**Example Response:**

```json
{
    "status": true,
    "message": "Broker Plan Retrieved Successfully!",
    "data": {
        "id": 10052,
        "name": "Gold Plan",
        "description": "Lorem ipsum dolor sit amet...",
        "category": "string",
        "state": "Published",
        "premium": 64500.00
    }
}
```

---
## 3 Broker Endorsements
### 3.1 `Create Endorsement`

- **Enums**
  - Marital Status
    - Single = 1
    - Married = 2
    - Divorced = 3
    - Widowed = 4
    - SingleParents = 5

  - Payment Plan
    - Annually    = 1
    - Bi-annually = 2
    - Quarterly   = 4
    - Monthly     = 12

  - Genotype
    - AA = 1
    - AO = 2
    - BB = 3
    - BO = 4
    - OO = 5
    - AS = 6

  - Blood Group
    - "A+" = 1
    - "O+" = 2
    - "B+" = 3
    - "AB+" = 4
    - "A-" = 5
    - "O-" = 6
    - "B-" = 7
    - "AB-" = 8

  - Religion
    - Islam = 1
    - Christianity = 2
    - Hinduism = 3
    - Buddhism = 4
    - Other = 5

  - Relation
    - Spouse = "Spouse"
    - Son = "Son"
    - Sibling = "Sibling"
    - Daughter = "Daughter"
    - Mother = "Mother"
    - Father = "Father"
    - Grand Father = "Grand Father"
    - Grand Mother = "Grand Mother"
    - Cousin = "Cousin"
    - Friend = "Friend"

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/endorsement/add`
- **Request Body**:
  - `reference` (required).
  - `plan_id` (required).
  - `policyId` (required).
  - `customer` (type: object).
    - `firstName` (type: string, required) - First name.
    - `middleName` (type: string) - Middle name.
    - `lastName` (type: string, required) - Last name.
    - `maidenName` (type: string) - Maiden name.
    - `_email` (type: string) - Email.
    - `dob` (type: date, required) - Date of birth.
    - `gender` (type: string, required) - Gender. Allowed values: 'Male', 'Female'.
    - `phone` (type: string, required) - Phone number. Must be a valid phone number.
    - `relation` (type: string, required) - Relationship status. Allowed values: [List of allowed relations].
    - `maritalStatus` (type: number, required) - Marital status. Allowed values: [List of allowed marital statuses].
    - `country` (type: number, required) - Country code.
    - `state` (type: number, required) - State code.
    - `city` (type: number, required) - City code.
    - `address` (type: string, required) - Residential address.
    - `genotype` (type: number) - Genotype. Allowed values: [List of allowed genotypes].
    - `bloodGroup` (type: number) - Blood group. Allowed values: [List of allowed blood groups].
    - `religion` (type: number) - Religion. Allowed values: [List of allowed religions].
    - `employeeId` (type: string) - Employee ID.
    - `nationalId` (type: string) - National ID.
    - `ninNumber` (type: string) - NIN (National Identification Number).
    - `height` (type: number) - Height.
    - `weight` (type: number) - Weight.
    - `preExistingCondition` (type: number) - Indicates if the customer has pre-existing conditions. Allowed values: 1 (true) or 0 (false).
    - `condition` (type: Array) - Array of pre-existing condition IDs. Include pre-existing condition IDs here if `preExistingCondition` is true.

  #### Notes:
  - If `preExistingCondition` is set to 1 (true), include an array of pre-existing condition IDs in the `condition` field to specify the customer's pre-existing conditions.
  - If `preExistingCondition` is set to 0 (false), the `condition` field can be omitted.

**Example Usage:**

```http
POST /api/v2/endorsement/add
```

**Example Response:**

```json
  {
    "status": true,
    "data": {
        "success": 1,
        "message": "Endorsement created successfully.",
        "membersDetails": {
            "planId": "85",
            "memberId": "100004976",
            "policyId": "2058",
            "firstName": "John",
            "lastName": "Doe",
            ...(all other customer data provided during onboarding)
        }
    }
  }
```
---

### 3.2 `Get pre-existing condition list`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/conditions?query={name}`
- **Query Parameters:**
  - `query` - Specify a Search text to filter conditions by the condition name

**Example Usage:**

```http
GET /api/v2/conditions?query=abdominal
```

**Example Response:**

```json
{
    "statusCode": 200,
    "message": "Successful!",
    "data": [
        {
            "id": "651",
            "pec_name": "Abdominal pregnancy",
            "pec_chapter_id": "14",
            "pec_status": "1"
        }
    ]
}
```

---

### 3.3 `Delete Endorsement`

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/endorsement/remove`
- **Request Body**:
  - `memberId` (required).
  - `policyId` (required).

**Example Usage:**

```http
POST /api/v2/endorsement/remove
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
        "success": 1,
        "message": "Endorsement deleted successfully.",
        "memberId": "100004976",
        "policyId": "2058"
    }
  }
```

---

### 3.4 `Get Enrollee Status`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/enrollee-status`
- **Query Parameters:**:
  - `memberId` (required).

**Example Usage:**

```http
GET /api/v2/enrollee-status?memberId=100004976
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
      "success": 1,
      "message": "Member Status Details",
      "member": {
        "memberId": "100004976",
        "firstName": "John",
        "lastName": "Doe",
        ...(other customer data)
        "status": "Active"
      }
    }
 }
```

---

### 3.5 `Get Enrollee Plan`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/enrollee-plan`
- **Query Parameters:**:
  - `memberId` (required).

**Example Usage:**

```http
GET /api/v2/enrollee-plan?memberId=100004976
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
      "success": 1,
      "message": "Member Plan Details",
      "member": {
        "memberId": "100004976",
        "firstName": "John",
        "lastName": "Doe",
        ...(other customer data)
        "planId": "85",
        "planName": "Plan"
      }
    }
  }
```

---

### 3.6 `Get Enrollee Utilization`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/enrollee-utilization`
- **Query Parameters:**:
  - `memberId` (required).

**Example Usage:**

```http
GET /api/v2/enrollee-utilization?memberId=100004976
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
        "success": 1,
        "message": "Fetched Member Utilization Successfully.",
        "membersDetails": {
            "memberId": "100004976",
            "firstName": "John",
            "lastName": "Doe",
            ...(other customer data)
            "planName": "Plan",
            "provider": "HOSPITAL",
            "totalSpent": 999,
            "balance": 9999999
        }
    }
}
```

---

### 3.7 `Undo Deleted Member Endorsement`

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/endorsement/undo-delete`
- **Request Body:**:
  - `memberId` (required).

**Example Usage:**

```http
POST /api/v2/endorsement/undo-delete
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
      "success": 1,
      "message": "Re-endorsed Member Successfully.",
      "membersDetails": {
        "planId": "85",
        "memberId": "100004976",
        "policyId": "2058",
        "firstName": "John",
        "lastName": "Doe",
        ...(all other customer data provided during onboarding)
      }
    }
  }
```

---

### 3.8 `Upgrade/Downgrade Member Plan`

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/endorsement/update-member-plan`
- **Request Body:**:
  - `memberId` (required).
  - `oldPlanId` (required).
  - `newPlanId` (required).

**Example Usage:**

```http
POST /api/v2/endorsement/update-member-plan
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
        "success": 1,
        "message": "Re-endorsed Member Successfully.",
        "membersDetails": {
          "memberId": "100004976",
          "policyId": "2058",
          "planId": "85",
          "firstName": "John",
          "lastName": "Doe",
          ...(all other customer data provided during onboarding)
      }
    }
  }
```

---

---

### 3.9 `Update Enrollee Details`

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `GET /api/v2/endorsement/update-enrollee-details`
- **Request Body:**:
  - `memberId` (required).
  - `policyId` (required).
  - `customer` (type: object).
    - `firstName` (type: string, required) - First name.
    - `middleName` (type: string) - Middle name.
    - `lastName` (type: string, required) - Last name.
    - `maidenName` (type: string) - Maiden name.
    - `_email` (type: string) - Email.
    - `dob` (type: date, required) - Date of birth.
    - `gender` (type: string, required) - Gender. Allowed values: 'Male', 'Female'.
    - `phone` (type: string, required) - Phone number. Must be a valid phone number.
    - `address` (type: string, required) - Residential address.

**Example Usage:**

```http
POST /api/v2/endorsement/update-member-plan
```

**Example Response:**

```json
 {
    "status": true,
    "data": {
        "success": 1,
        "message": "Member Information Updated Successfully.",
        "membersDetails": {
            "memberId": "100004976",
            "policyId": "2058",
            "firstName": "Jonny",
            "middleName": "",
            "maidenName": "",
            "lastName": "Doe",
            "email": "",
            "age": "22",
            "dob": "10-01-1962",
            "relation": "Self"
        }
    }
}
```

---

### 3.10 `Get Enrollee Auth Status`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/enrollee-auth-status`
- **Query Parameters:**:
  - `memberId` (required).

**Example Usage:**

```http
GET /api/v2/enrollee-auth-status?memberId=100004976
```

**Example Response:**

```json
{
  "status": true,
  "data": {
    "success": 1,
    "message": "Member Auth Status Details.",
    "member_auth": {
      "auth_number": "3000XXXXX",
      "message": "Dear Johm  Doe, an auth request 3000XXXXX from null, 1999-01-01 has been submitted for approval."
    }
  }
}
```

---

### 3.7 `Undo Multiple Deleted Member Endorsement`

- **Auth:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/endorsement/undo-delete`
- **Request Body:**:
  - `memberIds` (required).

**Example Usage:**

```http
POST /api/v2/endorsement/multi-undo-delete
```

**Example Request:**
```json
{
  "memberIds": [
      "10000XXXX",
      "10000XXXX",
      "10000XXXX"
  ]
}
```

**Example Response:**

```json
  {
    "status": true,
    "data": {
      "success": 1,
      "message": "Processed members successfully.",
      "membersDetails": {
        "processed_members": [
          {
              "memberId": "10000XXXX",
              "planId": "85",
              "planName": "Jade",
              "policyId": "2058",
              "firstName": "John",
              "middleName": "",
              "lastName": "Doe",
              ...(all other customer data provided during onboarding)
          },
          {
              "memberId": "10000XXXX",
              "planId": "85",
              "planName": "Jade",
              "policyId": "2058",
              "firstName": "John",
              "middleName": "",
              "lastName": "Doe",
              ...(all other customer data provided during onboarding)
          },
          {
              "memberId": "10000XXXX",
              "planId": "85",
              "planName": "Jade",
              "policyId": "2058",
              "firstName": "John",
              "middleName": "",
              "lastName": "Doe",
              ...(all other customer data provided during onboarding)
          }
        ]
      },
      "failed_members": [
          {
              "memberId": "10000XXX",
              "error": "Member not found or already active."
          }
      ]
    }
  }
```

---

## 4 Vendor API Config

### 4.1 `Update notification webhook URL`

- **Auth:** `Auth Type II`
- **HTTP Method:** `PATCH`
- **Endpoint:** `/api/v2/vendors/config`
- **Request Body**:
  - `webhookURL` (type: URI, required) - Vendor's notification webhookURL.
  - `webhookAccessKey` (type: string) - Access key for request authorization

**Example Usage:**

```http
PATCH /api/v2/vendors/config
```

**Example Response:**

```json
{
  "status": true,
  "message": "Configuration updated successfully!"
}
```

---

## 5 Vendor Webhook Notification Events DataTypes

### 5.1 `Create Endorsement event datatype`

- **HTTP Method:** `POST`
- **Endpoint:** `{{vendor's specified webhook URL}}`

**Example Event:**

```http
POST {{vendor's specified webhook URL}}
```

**Example Event Payload:**

```json
{
  "event": "added-policy-member",
  "accessKey": "{{Vendor's specified access key}}",
  "data": {
    "planId": "85",
    "memberId": "100004976",
    "policyId": "2058",
    "firstName": "John",
    "lastName": "Doe",
    ...(all other customer data provided during onboarding)
  }
}
```

### 5.2 `Undo Deleted Member Endorsement event datatype`

- **HTTP Method:** `POST`
- **Endpoint:** `{{vendor's specified webhook URL}}`

**Example Event:**

```http
POST {{vendor's specified webhook URL}}
```

**Example Event Payload:**

```json
{
  "event": "re-endorsed-member",
  "accessKey": "{{Vendor's specified access key}}",
  "data": {
    "planId": "85",
    "memberId": "100004976",
    "policyId": "2058",
    "firstName": "John",
    "lastName": "Doe",
    ...(all other customer data provided during onboarding)
  }
}
```

---

## 6 Misc APIs

### 6.1 `Get Regions/States`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/miscs/states`

**Example Usage:**

```http
GET /api/v2/miscs/states
```

**Example Response:**

```json
{
  "status": true,
  "message": "States/Regions Retrieved Successfully!",
  "data": [
    {
      "id": 15,
      "name": "Abuja (FCT)",
      "country_id": 162
    },
  ]
}
```

### 6.2 `Get Cities`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/miscs/cities?state_id={id}`
- **Query Parameters:**
  - `state_id` (required) - The state id retrieved from section 5.1 above.

**Example Usage:**

```http
GET /api/v2/miscs/cities?state_id=15
```

**Example Response:**

```json
{
  "status": true,
  "message": "Cities Retrieved Successfully!",
  "data": [
    {
      "id": 271,
      "name": "Gwagwalada",
      "state_id": 15,
    },
  ]
}
```

### 6.3 `Get providers list`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/miscs/providers-list?provider_type={type}&state_id={id}&city_id={cid}&plan_id={plan_id}`
- **Query Parameters:**
  - `state_id` - The state id retrieved from the GET States/Regions API in section 5.1 above.
  - `city_id` - The city id retrieved from the GET Cities API above.
  - `provider_type` - Valid values specified below.
  - `search_text` - Specify a Search text to filter providers by the provider name.
  - `plan_id` - Vendors plan_id.

- **Provider Types**
```
  1	= Hospital
  2	= Pharmacy
  3	= Optical / Ophthalmology
  5	= Physiotherapist
  6	= Dentist
  7 = Polyclinic
  8 = Doctor
  9 = Ambulatory Center
  10 = Diagnostic Centres
  11 = Radiology
  12 = Medical Supplies Company
  13 = Gym/Fitness Centres
  14 = Mental Wellbeing
  15 = Child Care
  16 = Maternal Health
  17 = Emergency Service
  18 = Public Health Facility
```

**Example Usage:**

```http
GET /api/v2/miscs/providers-list?provider_type=1&state_id=10&city_id=115&search_text=ABC Health&plan_id=85
```

**Example Response:**

```json
{
  "status": true,
  "message": "Providers List Retrieved Successfully!",
  "data": [
    {
      "name": "ABC Health",
      "category": "category",
      "email": "abc@health.com",
      "phone": "+234909291111111",
      "code": "ABC-HEALTH",
      "addressLine1": "Address line 1",
      "addressLine2": "Address line 2",
      "state": "Lagos",
      "city": "Ikeja",
      "longitude": "0.000000024",
      "latitude": "0.000232108",
      "rating": "5"
    },
  ]
}
```
