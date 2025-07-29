## Bastion Policy Web API Documentation

> **This is the official documentation for the Bastion Policy Web API.**
>
> **Important:** This API is **deprecated** and shouldn't be utilized

---

## Docs

**Base URLs:** 
 - **Dev:** https://demo-web.bastionhmo.com/
 - **Prod:** https://live-web.bastionhmo.com/

## Auth


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
 

### `Generate auth token`

- **Auth Type:** `Auth Type I`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v1/generateAuthToken`
- **Request Body:**
  - `vendorId` (required) - ID of Vendor.
  - `expiresIn` (type: numbe4, required) - Time token expires in seconds.

**Example Usage:**

```http
POST /api/v1/generateAuthToken

{
  "vendorId": "exampleVendorId",
  "expiresIn": 3600
}
```

**Notes:**

- Replace **_Base64EncodedCredentials_** with the Base64 encoding of the vendor's username and password in the format `username:password`.

**Example Response:**

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

### `Change Password`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v1/change-password/:token`
- **Parameters:**
  - `token` (required) - Authentication token.
- **Request Body**:
  - `oldPassword` (required) - New password for the vendor.
  - `newPassword` (required) - New password for the vendor.
  - `newConfirmPassword` (required) - New password for the vendor.

**Example Usage:**

```http
POST /api/v1/change-password/:token
{
  "oldPassword": "formerSecurePassword",
  "newPassword": "newSecurePassword",
  "newConfirmPassword": "newSecurePassword"
}
```

**Example Response:**

```json
{
  "status": 200,
  "body": {
    "success": true,
    "message": "Password Successfully updated!"
  }
}
```

---

### `Reset password`

- **Auth Type:** `Auth Type 0 - No Auth`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/password-reset/:email`
- **Parameters:**
  - `email` (required) - Email of the vendor.

**Example Usage:**

```http
GET /api/v1/password-reset/:email
```

**Example Response:**

```json
{
  "statusCode": 200,
  "message": "Successful!"
}
```

---

## Broker

### `Get broker plans`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/plans/`

**Example Usage:**

```http
GET /api/v1/plans/
```

**Example Response:**

```json
{
  "statusCode": 200,
  "message": "Broker Plans Retrieved Successfully!",
  "data": [
    {
      "name": "Gold Plan",
      "description": "Lorem ipsum dolor sit amet...",
      "category": "string",
      "state": "Published"
    },
    {
      "name": "Silver Plan",
      "description": "Lorem ipsum dolor sit amet...",
      "category": "string",
      "state": "Draft"
    },
    {
      "name": "Silver Plan",
      "description": "Lorem ipsum dolor sit amet...",
      "category": "string",
      "state": "Published"
    }
  ]
}
```

---

### `Get broker plan`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/plans/:id`
- **Parameters:**
  - `id` (required) - ID of the vendor.

**Example Usage:**

```http
GET /api/v1/plans/:id
```

**Example Response:**

```json
{
    "statusCode": 200,
    "message": "Broker Plan Retrieved Successfully!",
    "data": {
        "name": "Gold Plan",
        "description": "Lorem ipsum dolor sit amet...",
        "category": "string",
        "state": "Published" | "Draft"
    }
}
```

---

### `Get member details`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/member-details/:reference`
- **Parameters:**
  - `reference` (required) - Member reference.

**Example Usage:**

```http
GET /api/v1/webhook/vendors/member-details/:reference
```

**Example Response:**

```json
{
    "statusCode": 200,
    "message": "Member Details Retrieved Successfully!",
    "data": {
        "firstName": "John",
        "lastName": "Doe",
        "memberGroup": string,
        "state": string
    }
}
```

---

### `Get policy by paycode`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/get-policy/:payCode`
- **Parameters:**
  - `payCode` (required) - Payment code of the policy.

**Example Usage:**

```http
GET /api/v1/webhook/vendors/get-policy/:payCode
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

### `Get policy details`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/policy-details/:reference`
- **Parameters:**
  - `reference` (required) - Policy reference.

**Example Usage:**

```http
GET /api/v1/webhook/vendors/policy-details/:reference
```

**Example Response:**

```json
{
    "statusCode": 200,
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

### `Notify payment`

- **Auth Type:** `Auth Type II`
- **HTTP Method:** `POST`
- **Endpoint:** `/api/v1/webhook/vendors/:gid`
- **Parameters:**
  - `gid` (optional) - ID of the vendor.
- **Request Body**:
  - `reference` (type: boolean, required) - Payment reference.
  - `name` (required) - Name of vendor.
  - `vendorId` (required) - ID of the vendor.
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
  - `dependants` (type: customer[])

**Example Usage:**

```http
POST /api/v1/webhook/vendors/:gid?
```

**Example Response:**

```json
  {
    "status": true,
    "message": "sucessfully onboarded"
  }
```

---
