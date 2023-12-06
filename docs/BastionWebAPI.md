## Docs

**Base URLs:** 
 - **Dev:** https://demo-web.bastionhmo.com/
 - **Prod:** https://live-web.bastionhmo.com/

## Auth

### `Generate auth token`

- **HTTP Method:** `POST`
- **Endpoint:** `/api/v1/generateAuthToken`
- **Request Body:**
  - `vendorId` (required) - ID of Vendor.
  - `expiresIn` (type: numbe4, required) - Time token expires in seconds.
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Basic Base64EncodedCredentials

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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/plans/`
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
GET /api/v1/plans/
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/plans/:id`
- **Parameters:**
  - `id` (required) - ID of the vendor.
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
GET /api/v1/plans/:id
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/member-details/:reference`
- **Parameters:**
  - `reference` (required) - Member reference.
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
GET /api/v1/webhook/vendors/member-details/:reference
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/get-policy/:payCode`
- **Parameters:**
  - `payCode` (required) - Payment code of the policy.
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
GET /api/v1/webhook/vendors/get-policy/:payCode
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

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

- **HTTP Method:** `GET`
- **Endpoint:** `/api/v1/webhook/vendors/policy-details/:reference`
- **Parameters:**
  - `reference` (required) - Policy reference.
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
GET /api/v1/webhook/vendors/policy-details/:reference
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

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
- **Request Headers:**
  - `Content-Type`: application/json
  - `Authorization`: Bearer `{{token}}`

**Example Usage:**

```http
POST /api/v1/webhook/vendors/:gid?
```

**Notes:**

- Replace `{{token}}` with the response from the [`generateAuthToken`](#generate-auth-token) API.

**Example Response:**

```json
  {
    "status": true,
    "message": "sucessfully onboarded"
  }
```

---

### `Notify payment - Policy Payment`

- **HTTP Method:** `POST`
- **Endpoint:** `/policy-payments/callback`

**Example Usage:**

```http
POST /policy-payments/callback
```

**Example Response:**

```json
{
  "status": 200
}
```

---
