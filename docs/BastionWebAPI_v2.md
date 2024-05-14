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

## 2 Broker

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

### 2.3 `Get member details`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/member-details/:reference`
- **Parameters:**
  - `reference` (required) - Member reference.

**Example Usage:**

```http
GET /api/v2/vendors/member-details/:reference
```

**Example Response:**

```json
{
    "status": true,
    "message": "Member Details Retrieved Successfully!",
    "data": {
        "firstName": "John",
        "lastName": "Doe",
        "memberGroup": "",
        "state": ""
    }
}
```

---

### 2.4 `Get policy by paycode`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/get-policy/:payCode`
- **Parameters:**
  - `payCode` (required) - Payment code of the policy.

**Example Usage:**

```http
GET /api/v2/vendors/get-policy/:payCode
```

**Example Response:**

```json
{
    "statusCode": 200,
    "message": "Policy Retrieved Successfully!",
    "data": {
        "amount": string,
        "currency": "NGN",
        "trx_ref": string,
        "description": string,
        "customer": {
          "firstName": "John",
          "lastName": "Doe",
          "email": "john@doe.com"
        }
    }
}
```

---

### 2.5 `Get policy details`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/policy-details/:reference`
- **Parameters:**
  - `reference` (required) - Policy reference.

**Example Usage:**

```http
GET /api/v2/vendors/policy-details/:reference
```

**Example Response:**

```json
{
    "status": true,
    "message": "Policy Details Retrieved Successfully!",
    "data": {
      "id": '007',
      "firstName":string,
      "lastName":string,
      "email":string,
      "phone":string,
      "address":string,
      "amount":string,
      "plan_id": string,
      "dateOfBirth": DateTime,
      "preExistingConditions": json,
      "status": string,
      "startDate": DateTime,
      "nextPaymentDate": DateTime,
      "published_at": DateTime,
      "created_at": DateTime,
      "updated_at": DateTime,
    }
}
```

---

### 2.6 `Notify payment`

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
- **Endpoint:** `/api/v2/webhook/vendors`
- **Request Body**:
  - `reference` (type: string, required) - Payment reference.
  - `plan_id` (required).
  - `paymentPlan` (type: number, required)
  - `isReactivatePolicy` (type: boolean).
  - `narration`
  - `customer` (type: object)
    - `firstName` (type: string, required) - First name.
    - `middleName` (type: string) - Middle name.
    - `lastName` (type: string, required) - Last name.
    - `maidenName` (type: string) - Maiden name.
    - `dob` (type: date, required) - Date of birth.
    - `gender` (type: string) - Gender. Allowed values: 'Male', 'Female'.
    - `phone` (type: string, required) - Phone number. Must be a valid phone number.
    - `email` (type: string, required) - Email address. Must be a valid email.
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
    - `externalRef` (type: string) - The customer's external reference or ID that uniquely identifies the customer on the vendor's platform.
  - `dependants` (type: customer[])

**Example Usage:**

```http
POST /api/v2/webhook/vendors
```

**Example Response:**

```json
  {
    "status": true,
    "message": "Members onboarded sucessfully!",
    "data": {
      "membersDetails": [
        {
          "firstName": "John",
          "lastName": "Doe",
          ...(all other customer data provided during onboarding)
          "octohealthMemberId": "100020001",
          "externalRef": "VENDOR-A-012345",
        }
      ],
      "issuedPolicy": {
        "reference": "REF-190934134",
        "policyNo": "23409",
        "policyId": "POL-1248901",
        "status": "active",
        "startDate": "2024-01-31",
        "nextPaymentDate": "2025-01-29"
      }
    }
  }
```

---
## 3 Endorsements
### 3.1 `Create Endorsement`

- **Enums**
  -Section 2.6

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
  "statusCode": 200,
  "message": "Successful!",
  "data": {
      "memberId": "100004976",
      "firstName": "John",
      "lastName": "Doe",
      ...(other customer data)
      "status": "Active"
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
  "statusCode": 200,
  "message": "Successful!",
  "data": {
      "memberId": "100004976",
      "firstName": "John",
      "lastName": "Doe",
      ...(other customer data)
      "planId": "85",
      "planName": "Plan"
    }
  }
```

---

### 3.5 `Undo Deleted Member Endorsement`

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
    "statusCode": 200,
    "message": "Successful!",
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

### 5.1 `Issued Policy event datatype`

- **HTTP Method:** `POST`
- **Endpoint:** `{{vendor's specified webhook URL}}`

**Example Event:**

```http
POST {{vendor's specified webhook URL}}
```

**Example Event Payload:**

```json
{
  "event": "policy-issued",
  "accessKey": "{{Vendor's specified access key}}",
  "data": {
    "reference": "REF-190934134",
    "policyNo": "23409",
    "policyId": "POL-1248901",
    "status": "active",
    "startDate": "2024-01-31",
    "nextPaymentDate": "2025-01-29"
  }
}
```

### 5.2 `Member Onboard event datatype`

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
    "firstName": "John",
    "lastName": "Doe",
    ...(all other customer data provided during onboarding)
    "octohealthMemberId": "100020001",
    "externalRef": "VENDOR-A-012345",
  }
}
```

### 5.3 `Create Endorsement event datatype`

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

### 5.3 `Undo Deleted Member Endorsement event datatype`

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
- **Endpoint:** `/api/v2/miscs/providers-list?provider_type={type}&state_id={id}&city_id={cid}`
- **Query Parameters:**
  - `state_id` - The state id retrieved from the GET States/Regions API in section 5.1 above.
  - `city_id` - The city id retrieved from the GET Cities API above.
  - `provider_type` - Valid values specified below.
  - `search_text` - Specify a Search text to filter providers by the provider name.

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
GET /api/v2/miscs/providers-list?provider_type=1&state_id=10&city_id=115&search_text=ABC Health
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
