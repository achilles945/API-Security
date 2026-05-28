# REST Testing Methodology

---

# Endpoint Mapping

Identify:

* API base paths
* versions
* administrative routes
* internal APIs
* undocumented endpoints

Discovery sources:

* frontend traffic
* JavaScript files
* mobile applications
* OpenAPI/Swagger
* archived responses
* API gateways
* error messages

Common targets:

```text id="7vx0vt"
/api/
/v1/
/v2/
/internal/
/admin/
/beta/
```

Testing:

```http id="m4y01p"
OPTIONS /api/users HTTP/1.1
```

Observe:

* allowed methods
* CORS behavior
* hidden functionality

---

# HTTP Method Testing

Compare behavior across:

* GET
* POST
* PUT
* PATCH
* DELETE

Example:

```http id="d8x27u"
GET /api/users/15 HTTP/1.1
```

```http id="8x2t0y"
PUT /api/users/15 HTTP/1.1
```

```http id="4v7gcz"
DELETE /api/users/15 HTTP/1.1
```

Observe:

* authorization differences
* validation differences
* status code differences
* response differences

Backend indicators:

* missing method authorization
* inconsistent middleware enforcement
* unsupported method exposure

---

# Authentication Analysis

Identify:

* session-based auth
* JWT
* OAuth
* API keys
* custom headers

Authentication locations:

* Authorization header
* cookies
* custom headers
* query parameters

Examples:

```http id="jvjlwm"
Authorization: Bearer TOKEN
```

```http id="n5p4pj"
Cookie: session=abc123
```

Testing targets:

* token reuse
* token expiration
* refresh token handling
* logout invalidation
* MFA enforcement
* brute force protections

Observe:

* token validation behavior
* session invalidation behavior
* authentication persistence
* response consistency

---

# Authorization Testing

Use multiple accounts with different privilege levels.

Compare:

* accessible endpoints
* returned fields
* allowed actions
* available methods
* object visibility

Example:

```http id="pjlwm7"
GET /api/users/15 HTTP/1.1
Authorization: Bearer USER_A
```

```http id="njlwm1"
GET /api/users/16 HTTP/1.1
Authorization: Bearer USER_A
```

Observe:

* unauthorized object access
* response structure differences
* status code inconsistencies
* hidden data exposure

Backend indicators:

* missing ownership validation
* weak ORM filtering
* missing middleware authorization

---

# Object Reference Testing

Identify:

* numeric IDs
* UUIDs
* slugs
* relationship identifiers

Examples:

```http id="2v6eqk"
GET /api/orders/1001
```

```http id="yok3tp"
GET /api/invoices/INV-8821
```

Testing methodology:

* increment IDs
* decrement IDs
* replace UUIDs
* access cross-tenant objects
* compare responses across accounts

Observe:

* unauthorized access
* field differences
* authorization inconsistencies
* object existence leakage

---

# Parameter Discovery

Identify:

* query parameters
* JSON properties
* hidden fields
* internal flags
* nested objects

Sources:

* frontend JavaScript
* mobile traffic
* Swagger/OpenAPI
* error messages
* response metadata

Examples:

```http id="gdyzfd"
GET /api/users?role=admin
```

```json id="k0z5sd"
{
  "is_admin": true
}
```

Testing targets:

* hidden parameters
* role fields
* internal flags
* nested objects
* relationship objects

Observe:

* authorization changes
* response differences
* validation changes
* object state changes

---

# Parameter Pollution Testing

Examples:

```http id="rk6ffr"
GET /api/users?id=1&id=2
```

```http id="cc9xbn"
GET /api/orders?role=user&role=admin
```

Testing methodology:

* duplicate parameters
* conflicting parameters
* nested parameters
* array injection
* parameter type confusion

Observe:

* parser precedence
* backend parameter selection
* validation inconsistencies
* authorization inconsistencies

Backend indicators:

* parser differential behavior
* framework-specific parameter handling
* proxy/backend inconsistencies

---

# JSON Body Manipulation

Example request:

```http id="4h5pmt"
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "username": "alice"
}
```

Add hidden fields:

```json id="mjlwm2"
{
  "username": "alice",
  "role": "admin",
  "verified": true
}
```

Testing targets:

* role fields
* internal flags
* ownership fields
* pricing fields
* workflow state fields

Observe:

* reflected fields
* database changes
* authorization changes
* privilege escalation

Backend indicators:

* Mass Assignment
* unsafe object mapping
* hidden property exposure

---

# Content-Type Testing

Replay identical requests using different content types.

Examples:

```http id="9okp8x"
Content-Type: application/json
```

```http id="jjlwm4"
Content-Type: application/xml
```

```http id="zwjlwm"
Content-Type: multipart/form-data
```

Observe:

* validation differences
* parser differences
* authorization differences
* deserialization differences

Backend indicators:

* parser inconsistencies
* validation bypass
* alternate processing paths

---

# Header Manipulation Testing

Headers to test:

```http id="6zjlwm"
X-Forwarded-For
X-Original-URL
X-Rewrite-URL
X-User-ID
X-Internal-Request
```

Examples:

```http id="cbjlwm"
X-Forwarded-For: 127.0.0.1
```

```http id="jlwmn9"
X-Original-URL: /admin
```

Observe:

* route rewriting
* authorization bypass
* internal route exposure
* IP-based trust behavior

Backend indicators:

* proxy trust issues
* gateway trust issues
* internal header trust

---

# API Version Testing

Common version patterns:

```text id="jlwm6o"
/v1/
/v2/
/beta/
/internal/
```

Testing methodology:

* compare endpoint behavior across versions
* compare authorization
* compare response fields
* compare validation logic

Observe:

* deprecated functionality
* weaker authorization
* hidden functionality
* inconsistent validation

---

# File Upload Testing

Common upload endpoints:

```text id="jlwm4t"
/upload
/avatar
/import
/documents
```

Testing targets:

* file extensions
* MIME types
* polyglot files
* filename handling
* metadata handling

Examples:

```http id="3jlwm8"
Content-Type: multipart/form-data
```

Observe:

* extension validation
* MIME validation
* file storage paths
* image processing behavior

Backend indicators:

* unrestricted upload
* parser abuse
* file processing vulnerabilities

---

# Response Analysis

Analyze:

* response fields
* hidden metadata
* authorization indicators
* serializer behavior
* object relationships

Example:

```json id="jlwmf5"
{
  "id": 1,
  "role": "admin",
  "internal": true
}
```

Observe:

* hidden fields
* authorization flags
* internal identifiers
* relationship exposure
* debug information

Compare responses across:

* users
* roles
* methods
* content types
* API versions

---

# Error Handling Analysis

Generate:

* malformed JSON
* invalid object IDs
* unsupported methods
* invalid types
* malformed headers

Examples:

```json id="jlwmw2"
{
  "id": "AAAA"
}
```

Observe:

* stack traces
* ORM errors
* framework details
* internal paths
* validation logic

Backend indicators:

* framework fingerprinting
* ORM fingerprinting
* parser behavior
* internal architecture disclosure

---

# Rate Limiting Analysis

Testing targets:

* login endpoints
* OTP endpoints
* password reset
* invitation systems
* search functionality

Testing methodology:

* burst requests
* parallel requests
* distributed requests
* token rotation
* IP rotation

Observe:

* rate-limit headers
* blocking behavior
* temporary bans
* inconsistent enforcement

Backend indicators:

* missing rate limiting
* per-IP only enforcement
* endpoint-specific gaps

---

# Race Condition Testing

Common targets:

* coupon redemption
* balance transfers
* checkout flows
* inventory updates
* OTP verification

Testing methodology:

* parallel requests
* replay requests
* delayed requests
* duplicated requests

Example:

```http id="jlwm12"
POST /api/coupons/redeem HTTP/1.1
```

Send simultaneously.

Observe:

* duplicated transactions
* balance inconsistencies
* state desynchronization
* duplicate resource creation

Backend indicators:

* missing transaction locking
* missing atomic operations
* async processing flaws

---

# Business Logic Testing

Identify workflows:

```text id="jlwm5d"
Cart
→ Checkout
→ Payment
→ Confirmation
```

Testing methodology:

* reorder requests
* skip steps
* replay requests
* manipulate state values
* manipulate pricing

Observe:

* state inconsistencies
* skipped validation
* unauthorized transitions
* duplicate operations

Backend indicators:

* workflow trust assumptions
* missing state validation
* client-side state trust

---

# Gateway & Reverse Proxy Testing

Common targets:

* internal routes
* rewritten URLs
* forwarded headers
* unsupported methods

Testing methodology:

* conflicting headers
* malformed requests
* duplicate headers
* path normalization bypass

Observe:

* route inconsistencies
* proxy/backend parsing differences
* internal route exposure

Backend indicators:

* HTTP desynchronization
* proxy trust abuse
* gateway bypass

---

# Caching Analysis

Testing targets:

* authenticated responses
* role-based content
* profile endpoints
* dashboard APIs

Observe:

* cache headers
* response reuse
* cross-user content exposure
* stale authorization state

Backend indicators:

* authenticated response caching
* cache key confusion
* authorization bypass through cache
