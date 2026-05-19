# OWASP API Security Top 10

OWASP API Security Top 10 is a list of the most critical vulnerabilities commonly found in APIs.

Unlike traditional web vulnerabilities:
- API vulnerabilities heavily focus on authorization
- APIs directly expose backend functionality
- APIs expose raw objects and business logic
- APIs are commonly consumed by mobile apps, frontend frameworks, and third-party services

Most API vulnerabilities occur because:
- backend trusts client-controlled input
- authorization checks are missing
- hidden functionality is exposed through APIs
- APIs expose internal object structures directly

The OWASP API Top 10 helps identify:
- common API attack surfaces
- backend authorization flaws
- business logic weaknesses
- insecure API designs

---

# API1 — Broken Object Level Authorization (BOLA)

## Overview

Broken Object Level Authorization occurs when backend fails to validate whether a user is allowed to access a specific object.

API endpoints commonly expose object identifiers directly.

Examples:

```http
GET /api/users/124
GET /api/orders/55
GET /api/messages/90
```

Backend may verify:
- user is authenticated

but fail to verify:
- whether object belongs to that user

This allows attackers to access:
- other users' data
- sensitive objects
- unauthorized resources

BOLA is one of the most common API vulnerabilities because APIs heavily expose:
- object IDs
- database records
- internal resources

---

## How BOLA Happens Internally

Backend typically performs:

```text
SELECT * FROM users WHERE id=124
```

Problem:
- backend trusts user-controlled object ID
- ownership validation missing

Instead of validating:

```text
Does current authenticated user own object 124?
```

backend directly returns object.

---

## Common Object Identifiers

Examples:
- numeric IDs
- UUIDs
- GUIDs
- usernames
- email addresses
- account numbers

Example UUID:

```text
550e8400-e29b-41d4-a716-446655440000
```

---

## Common Vulnerabilities

- IDOR
- Horizontal Privilege Escalation
- Unauthorized Object Access
- Insecure Direct Object Reference

---

## Common Targets

- User profiles
- Orders
- Payment details
- Messages
- Invoices
- Tickets
- API keys
- Files

---

## Testing Methodology

### Step 1 — Identify Object References

Look for:
- IDs in URLs
- IDs in JSON bodies
- UUIDs
- account numbers

Example:

```http
GET /api/users/124
```

---

### Step 2 — Change Object Identifiers

Modify object references.

Example:

```http
GET /api/users/125
```

Observe whether:
- another user's data becomes accessible
- response changes
- authorization errors appear

---

### Step 3 — Compare Different User Roles

Use multiple accounts:
- normal user
- privileged user
- administrator

Compare:
- accessible endpoints
- returned objects
- response fields
- allowed actions

Check whether backend properly restricts:
- object ownership
- object visibility
- object modification

---

### Step 4 — Test Different HTTP Methods

Example:

```http
GET /api/orders/100
```

Test:
- PUT
- PATCH
- DELETE

Check whether backend restricts:
- modification
- deletion
- sensitive actions

---

## Common Impacts

- Sensitive data exposure
- Unauthorized actions
- Account takeover
- Data manipulation
- Financial impact

---

# API2 — Broken Authentication

## Overview

Broken Authentication occurs when authentication mechanisms are:
- weak
- improperly implemented
- improperly validated

Authentication systems verify:
- user identity

If authentication fails:
- attackers may impersonate users
- hijack sessions
- bypass login systems
- access protected APIs

---

## Common Authentication Mechanisms

APIs commonly use:
- JWT
- Session cookies
- OAuth
- API keys
- Access tokens
- Refresh tokens

---

## Common Vulnerabilities

- Weak JWT secrets
- JWT algorithm confusion
- Missing signature validation
- Session fixation
- Token leakage
- Weak password reset
- OAuth misconfiguration
- Predictable tokens

---

## JWT Authentication

JWT structure:

```text
HEADER.PAYLOAD.SIGNATURE
```

Payload commonly contains:
- user ID
- roles
- expiration

JWT is:
- base64 encoded
- digitally signed

Backend validates:
- token signature
- expiration
- claims

Weak validation may allow:
- token forgery
- privilege escalation

---

## Testing Methodology

### Step 1 — Identify Authentication Mechanism

Check:
- Authorization headers
- cookies
- tokens
- OAuth redirects

Examples:

```http
Authorization: Bearer TOKEN
```

```http
Cookie: sessionId=abc123
```

---

### Step 2 — Analyze JWT Tokens

Decode JWT and inspect:
- header
- payload
- algorithm
- expiration
- roles

Check for:
- weak algorithms
- missing validation
- exposed secrets

---

### Step 3 — Test Token Validation

Test:
- modified payloads
- expired tokens
- unsigned tokens
- reused tokens

Observe whether backend:
- rejects invalid tokens
- validates signatures correctly

---

### Step 4 — Analyze Session Handling

Check:
- session rotation
- predictable sessions
- logout invalidation
- session reuse

---

## Common Impacts

- Account takeover
- Authentication bypass
- Privilege escalation

---

# API3 — Broken Object Property Level Authorization

## Overview

Broken Object Property Level Authorization occurs when backend improperly exposes or processes object properties.

APIs commonly exchange JSON objects.

Example:

```json
{
  "id": 1,
  "username": "alice",
  "role": "user"
}
```

Backend may expose:
- hidden fields
- internal properties
- sensitive attributes

or allow attackers to modify:
- restricted properties
- privileged fields

---

## How It Happens Internally

Modern backend frameworks often automatically map JSON fields into backend objects.

Example:

```json
{
  "isAdmin": true
}
```

If backend does not explicitly restrict properties:
- sensitive fields may become modifiable

---

## Common Vulnerabilities

- Mass Assignment
- Excessive Data Exposure
- Hidden Property Manipulation

---

## Excessive Data Exposure

Backend returns more data than required.

Example:

```json
{
  "id": 1,
  "email": "alice@example.com",
  "passwordHash": "...",
  "internalRole": "admin"
}
```

Sensitive fields become exposed because:
- frontend filtering relied upon
- backend returns full object

---

## Testing Methodology

### Step 1 — Analyze Response Objects

Look for:
- hidden fields
- internal flags
- role values
- backend metadata

---

### Step 2 — Add Hidden Parameters

Add:
- role fields
- admin flags
- internal attributes

Example:

```json
{
  "role": "admin"
}
```

Check whether backend:
- accepts field
- updates property
- changes privileges

---

### Step 3 — Compare Role Responses

Use:
- normal user account
- administrator account

Compare:
- returned JSON fields
- accessible properties
- hidden attributes

Check whether:
- sensitive properties leak to low-privileged users

---

## Common Impacts

- Privilege escalation
- Sensitive data exposure
- Role manipulation

---

# API4 — Unrestricted Resource Consumption

## Overview

Unrestricted Resource Consumption occurs when backend fails to properly limit:
- request volume
- processing complexity
- resource usage

APIs may allow attackers to consume excessive:
- CPU
- memory
- bandwidth
- database resources

This can lead to:
- denial of service
- backend exhaustion
- infrastructure instability

---

## Common Vulnerabilities

- Missing rate limiting
- GraphQL query abuse
- File upload abuse
- Infinite pagination
- Resource exhaustion
- Race conditions

---

## Rate Limiting

Rate limiting controls:
- how many requests clients can send

Example protections:
- requests per minute
- IP-based throttling
- account-based limits

Missing rate limiting may allow:
- brute force
- OTP abuse
- flooding attacks

---

## Testing Methodology

### Step 1 — Test Repeated Requests

Send large number of requests rapidly.

Observe:
- throttling
- delays
- blocking
- HTTP 429 responses

---

### Step 2 — Test Concurrent Requests

Use parallel requests.

Check for:
- race conditions
- duplicate actions
- inconsistent backend state

---

### Step 3 — Test Large Payloads

Examples:
- huge JSON bodies
- large file uploads
- deeply nested GraphQL queries

Observe:
- response delays
- server crashes
- memory issues

---

## Common Impacts

- Denial of Service
- Infrastructure exhaustion
- Business logic abuse

---

# API5 — Broken Function Level Authorization

## Overview

Broken Function Level Authorization occurs when backend fails to restrict access to sensitive functionality.

Frontend applications commonly hide:
- admin panels
- privileged actions
- internal functionality

but backend APIs may still expose endpoints directly.

---

## How It Happens Internally

Backend validates:
- user authentication

but fails to validate:
- whether user can access specific function

Example:

```http
DELETE /api/users/1
```

Backend executes action because:
- endpoint reachable
- authorization missing

---

## Common Vulnerabilities

- Privilege escalation
- Unauthorized admin access
- Function-level authorization bypass

---

## Common Targets

- Admin APIs
- Internal endpoints
- User management
- Payment management
- Configuration APIs

---

## Testing Methodology

### Step 1 — Identify Sensitive Functions

Look for:
- admin endpoints
- internal APIs
- management functions

Examples:

```text
/api/admin
/api/internal
/api/config
```

---

### Step 2 — Test Low-Privilege Access

Use:
- normal user account

Attempt access to:
- admin endpoints
- privileged functions

Observe whether backend:
- properly blocks request
- returns sensitive data
- executes action

---

### Step 3 — Test Hidden HTTP Methods

Example:

```http
GET /api/users/1
```

Test:
- PUT
- PATCH
- DELETE

Check whether backend exposes:
- modification functionality
- deletion functionality
- administrative actions

---

## Common Impacts

- Privilege escalation
- Unauthorized admin access
- Sensitive functionality exposure