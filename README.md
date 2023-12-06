## Docs

**Base URL:** https://demo-web.bastionhmo.com/

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

- Replace **_Base64EncodedCredentials_** with the Base64 encoding of the vendor's username and password in the format **_username:password_**.

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

### `Get broker plans`

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
  "message": "Broker Plan Retrieved Successfully!",
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
        "state": "Published" | "Draft"
    }
}
```

---

### `Get policy by paycode`

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

### `Notify payment` <-----------

- **HTTP Method:** `POST`
- **Endpoint:** `/api/v1/webhook/vendors/:gid?`
- **Parameters:**
  - `gid` (optional) - ID of the vendor.
- **Request Body**:
  - `vendorId` (required) - ID of the vendor.
  - `name` (required) - Name of vendor.

**Example Usage:**

```http
POST /api/v1/webhook/vendors/:gid?
```

**Example Response:**

```json
{
    "statusCode": 200,
    "message": "Payment Notification Received!",
    "data":
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
  "statusCode": 200,
  "message": "Payment Notification Received for Policy!",
  "data": null
}
```

---
