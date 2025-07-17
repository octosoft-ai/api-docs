## Docs

**Base URLs:**

- **Dev:** https://demo-web.bastionhmo.com/
- **Prod:** https://live-web.bastionhmo.com/

## 1 Auth

**Auth Types:**

- 0.  No auth
- No authorization token is required

- I. `Api-key (Basic)`: `api-key: "Base64Encode(username:password)"`

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

## Members API

### 2.1 `Get Broker Enrollees (Paginated)`

- **Auth:** `Auth Type II`
- **HTTP Method:** `GET`
- **Endpoint:** `/api/v2/enrollee-all`
- **Parameters:**
  - `page` (optional) - Page number for pagination (default: 1)
  - `per_page` (optional) - Number of records per page (default: 20, max: 100)

**Example Usage:**

```http
GET /api/v2/enrollee-all&page=1&per_page=20
```

**Example Usage (Default Pagination):**

```http
GET /api/v2/enrollee-all
```

**Example Response (Success):**

```json
{
  "success": 1,
  "message": "Member List.",
  "policy_id": "12345",
  "total_members": 250,
  "members": [
    {
      "first_name": "John",
      "middle_name": "Michael",
      "last_name": "Smith",
      "mm_member_id": "MEM001",
      "phone": "+1234567890",
      "age": 35,
      "dob": "1988-05-15",
      "gender": "Male",
      "planName": "Premium Health Plan",
      "city": "FCT",
      "address": "123 Main Street",
      "state": "Abuja",
      "country": "Nigeria",
      "registration_date": "2023-01-15",
      "status": "Active"
    },
    {
      "first_name": "Jane",
      "middle_name": "Elizabeth",
      "last_name": "Doe",
      "mm_member_id": "MEM002",
      "phone": "+1234567891",
      "age": 28,
      "dob": "1995-08-22",
      "gender": "Female",
      "planName": "Basic Health Plan",
      "city": "Ikeja",
      "address": "456 Oak Avenue",
      "state": "Lagos",
      "country": "Nigeria",
      "registration_date": "2023-02-10",
      "status": "Active"
    }
  ],
  "pagination": {
    "current_page": 1,
    "per_page": 20,
    "total_records": 250,
    "total_pages": 13,
    "has_next": true,
    "has_prev": false,
    "from": 1,
    "to": 20
  }
}
```

**Example Response (Unauthorized):**

```json
{
  "success": 0,
  "message": "Unauthorized"
}
```

---