## Docs

**Base URLs:** 
 - **Dev:** https://demo-web.bastionhmo.com/
 - **Prod:** https://live-web.bastionhmo.com/

## 1 Auth

**Auth Types:**
  -  0. No auth
  ***Notes:***
    - No authorization token is required
    
  -  I. `Api-key (Basic)`  - `api-key: "Base64Encode(username:password)"`
  ***Notes:***
    - Vendor's credentials (username and password) will be shared during onboarding.
    - Example: `api-key: "YWRQxQA3dvcmtaW46UGFzc=="` - after the credentials have been encoded

  - II. `Api-key (Bearer)` - `api-key: {{auth_token}}`
  ***Notes:***
    - The `auth_token` can be retrieved as described in session 1.1 below
    - Example: `api-key: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwHMiLCJpYXQiOjE2NjAwNTcm92aWRlciI6InVuaWZpZWRfcGF5bWVudA1MTk2MjN9.AS13xG_MOXTKKfpYKGYHfArjPULtq4uMJ_CNpa_ACBp"`


**Request Headers:**
  - `Content-Type`: application/json
  - `api-key`: AuthType i or ii
 
 

### 1.1 `Generate auth token`

- **Auth Type:** `Auth Type I`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/auth/generateAuthToken`
- **Request Body:**
  - `expiresIn` (type: number, required) - Time token expires in seconds.

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

- **Auth Type:** `Auth Type 0 - No auth required`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/password-reset?email={email}`
- **Parameters:**
  - `email` (required) - Email of the vendor.

**Example Usage:**

```http
GET /api/v2/password-reset?email=johndoe@getnada.com
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

- **Auth Type:** `Auth Type II`
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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/plans/:id`
- **Parameters:**
  - `id` (required) - ID of the plan retrieved from the GET broker plans API.

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

- **Auth Type:** `Auth Type II`
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

- **Auth Type:** `Auth Type II`
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
    - Quaterly    = 4
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
    - Other = "Other"


- **Auth Type:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v2/webhook/vendors`
- **Request Body**:
  - `reference` (type: boolean, required) - Payment reference.
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
    - `externalRef` (type: number) - The customer's external reference or ID that uniquely identifies the customer on the vendor's platform.
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

## 3 Vendor API Config

### 3.1 `Update notification webhook URL`

- **Auth Type:** `Auth Type II`
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

## 4 Vendor Webhook Notification Events DataTypes

### 4.1 `Issued Policy event datatype`

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

### 4.2 `Member Onboard event datatype`

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


---

## 5 Providers List

### 5.1 `Get providers list`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/vendors/providers-list`

**Example Usage:**

```http
GET /api/v2/vendors/providers-list
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

```
