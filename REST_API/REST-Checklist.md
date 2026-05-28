# REST Checklist

---

# Recon

```text id="jlwmu1"
[ ] Identify API base paths
[ ] Identify API versions
[ ] Identify hidden endpoints
[ ] Identify admin routes
[ ] Identify internal APIs
[ ] Identify Swagger/OpenAPI exposure
[ ] Analyze frontend JavaScript
[ ] Analyze mobile application traffic
[ ] Analyze archived endpoints
[ ] Enumerate subdomains
```

Common locations:

```text id="jlwmr8"
/swagger
/openapi.json
/api-docs
/v1/
/v2/
/beta/
/internal/
```

---

# Endpoint Enumeration

```text id="1jlwmf"
[ ] Enumerate GET endpoints
[ ] Enumerate POST endpoints
[ ] Enumerate PUT endpoints
[ ] Enumerate PATCH endpoints
[ ] Enumerate DELETE endpoints
[ ] Test OPTIONS requests
[ ] Test unsupported methods
[ ] Identify undocumented methods
```

Testing:

```http id="jlwm7z"
OPTIONS /api/users HTTP/1.1
```

Observe:

* allowed methods
* hidden functionality
* CORS behavior

---

# Authentication Testing

```text id="tjlwm7"
[ ] Identify authentication mechanism
[ ] Test JWT validation
[ ] Test session handling
[ ] Test API key handling
[ ] Test refresh token handling
[ ] Test logout invalidation
[ ] Test MFA enforcement
[ ] Test brute-force protections
[ ] Test token expiration
[ ] Test token replay
```

Headers:

```http id="jlwm3g"
Authorization: Bearer TOKEN
```

Cookies:

```http id="jlwmr1"
Cookie: session=abc123
```

---

# Authorization Testing

```text id="jlwmq2"
[ ] Use multiple user roles
[ ] Compare accessible endpoints
[ ] Compare accessible objects
[ ] Compare response fields
[ ] Compare allowed methods
[ ] Compare authorization behavior
[ ] Test cross-account access
[ ] Test tenant isolation
[ ] Test admin functionality
[ ] Test hidden administrative endpoints
```

Testing:

```http id="xjlwm6"
GET /api/users/15 HTTP/1.1
Authorization: Bearer USER_A
```

```http id="zjlwm8"
GET /api/users/16 HTTP/1.1
Authorization: Bearer USER_A
```

Observe:

* unauthorized access
* field differences
* status code inconsistencies

---

# Object Reference Testing

```text id="qjlwm9"
[ ] Test sequential IDs
[ ] Test UUID manipulation
[ ] Test cross-tenant object access
[ ] Test relationship traversal
[ ] Test predictable identifiers
[ ] Test object enumeration
```

Examples:

```http id="9jlwm4"
GET /api/orders/1001
```

```http id="2jlwm7"
GET /api/users/15
```

---

# HTTP Method Testing

```text id="vjlwm0"
[ ] Compare GET/POST behavior
[ ] Compare PUT/PATCH behavior
[ ] Compare DELETE authorization
[ ] Test unsupported methods
[ ] Test method override headers
[ ] Test method-specific authorization
```

Headers:

```http id="mjlwm6"
X-HTTP-Method-Override: DELETE
```

Observe:

* authorization differences
* validation differences
* hidden functionality

---

# Parameter Testing

```text id="njlwm2"
[ ] Identify query parameters
[ ] Identify hidden parameters
[ ] Test duplicate parameters
[ ] Test nested parameters
[ ] Test array injection
[ ] Test parameter pollution
[ ] Test parameter type confusion
```

Examples:

```http id="jjlwm5"
GET /api/users?id=1&id=2
```

```http id="6jlwm3"
GET /api/orders?role=user&role=admin
```

Observe:

* parser behavior
* parameter precedence
* validation inconsistencies

---

# JSON Body Testing

```text id="rjlwm4"
[ ] Test hidden fields
[ ] Test role fields
[ ] Test internal flags
[ ] Test ownership fields
[ ] Test nested objects
[ ] Test relationship objects
[ ] Test type confusion
```

Example:

```json id="7jlwm1"
{
  "role": "admin",
  "verified": true
}
```

Observe:

* reflected fields
* privilege changes
* database changes
* authorization changes

---

# Content-Type Testing

```text id="wjlwm8"
[ ] Test application/json
[ ] Test application/xml
[ ] Test multipart/form-data
[ ] Test application/x-www-form-urlencoded
[ ] Compare parser behavior
[ ] Compare validation behavior
[ ] Compare authorization behavior
```

Examples:

```http id="5jlwm9"
Content-Type: application/json
```

```http id="yjlwm2"
Content-Type: application/xml
```

Observe:

* parser inconsistencies
* validation bypass
* alternate processing paths

---

# Header Testing

```text id="fjlwm6"
[ ] Test X-Forwarded-For
[ ] Test X-Original-URL
[ ] Test X-Rewrite-URL
[ ] Test X-User-ID
[ ] Test X-Internal-Request
[ ] Test duplicate headers
[ ] Test malformed headers
```

Examples:

```http id="ljlwm7"
X-Forwarded-For: 127.0.0.1
```

```http id="gjlwm0"
X-Original-URL: /admin
```

Observe:

* authorization bypass
* internal route exposure
* proxy trust behavior

---

# File Upload Testing

```text id="bjlwm3"
[ ] Test file extensions
[ ] Test MIME types
[ ] Test polyglot files
[ ] Test filename handling
[ ] Test metadata handling
[ ] Test image processing
[ ] Test upload size limits
```

Endpoints:

```text id="sjlwm1"
/upload
/avatar
/import
/documents
```

Observe:

* extension validation
* MIME validation
* file storage paths
* processing behavior

---

# API Version Testing

```text id="hjlwm4"
[ ] Enumerate API versions
[ ] Compare old/new versions
[ ] Compare authorization logic
[ ] Compare response structures
[ ] Compare validation logic
[ ] Test deprecated endpoints
[ ] Test beta routes
```

Examples:

```text id="djlwm8"
/v1/
/v2/
/beta/
/internal/
```

---

# Response Analysis

```text id="cjlwm5"
[ ] Analyze response fields
[ ] Analyze hidden metadata
[ ] Analyze serializer leakage
[ ] Analyze authorization flags
[ ] Analyze object relationships
[ ] Analyze internal identifiers
```

Example:

```json id="ujlwm2"
{
  "id": 1,
  "role": "admin",
  "internal": true
}
```

---

# Error Handling Analysis

```text id="kjlwm6"
[ ] Send malformed JSON
[ ] Send invalid object IDs
[ ] Send invalid types
[ ] Send malformed headers
[ ] Send unsupported methods
[ ] Analyze stack traces
[ ] Analyze framework disclosure
[ ] Analyze ORM disclosure
```

Example:

```json id="pjlwm4"
{
  "id": "AAAA"
}
```

Observe:

* stack traces
* ORM errors
* internal paths
* parser behavior

---

# Rate Limiting Testing

```text id="zjlwm5"
[ ] Test login rate limiting
[ ] Test OTP brute force
[ ] Test password reset throttling
[ ] Test invitation abuse
[ ] Test distributed requests
[ ] Test token rotation bypass
[ ] Test IP rotation bypass
```

Observe:

* rate-limit headers
* blocking behavior
* inconsistent enforcement

---

# Race Condition Testing

```text id="ojlwm3"
[ ] Test parallel requests
[ ] Test duplicated requests
[ ] Test replay attacks
[ ] Test delayed requests
[ ] Test state desynchronization
[ ] Test duplicate transactions
```

Targets:

```text id="4jlwm9"
Coupon redemption
Balance transfers
Checkout flows
Inventory updates
OTP verification
```

---

# Business Logic Testing

```text id="qjlwm1"
[ ] Map application workflows
[ ] Reorder requests
[ ] Skip workflow steps
[ ] Replay completed requests
[ ] Manipulate pricing
[ ] Manipulate state transitions
[ ] Test duplicate operations
```

Workflow example:

```text id="ejlwm2"
Cart
→ Checkout
→ Payment
→ Confirmation
```

Observe:

* skipped validation
* invalid state transitions
* duplicate transactions
* pricing inconsistencies

---

# Gateway & Reverse Proxy Testing

```text id="tjlwm5"
[ ] Test route rewriting
[ ] Test internal route exposure
[ ] Test malformed requests
[ ] Test duplicate headers
[ ] Test conflicting headers
[ ] Test path normalization bypass
[ ] Test proxy/backend inconsistencies
```

Headers:

```http id="vjlwm7"
X-Original-URL
X-Rewrite-URL
```

Observe:

* internal path exposure
* gateway bypass
* authorization inconsistencies

---

# Cache Testing

```text id="njlwm8"
[ ] Test authenticated response caching
[ ] Test cache key behavior
[ ] Test cache poisoning
[ ] Test cache deception
[ ] Test cross-user response exposure
[ ] Test stale authorization state
```

Observe:

* cache headers
* response reuse
* authorization leakage

---

# Internal API Testing

```text id="jjlwm0"
[ ] Enumerate internal routes
[ ] Test internal headers
[ ] Test SSRF exposure
[ ] Test gateway bypass
[ ] Test internal token trust
[ ] Test service impersonation
```

Examples:

```text id="xjlwm3"
/internal/users
/internal/debug
/internal/admin
```
