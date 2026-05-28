# REST Attack Surface

---

# Endpoints

REST APIs expose functionality through URLs.

Examples:

```text id="bz4g4y"
/api/users
/api/orders
/api/payments
/api/admin/users
```

Endpoint exposure sources:

* frontend applications
* mobile applications
* JavaScript files
* OpenAPI/Swagger
* API gateways
* reverse proxies
* archived endpoints
* undocumented routes

Attack Surface:

* hidden endpoints
* deprecated endpoints
* internal APIs
* administrative functionality
* debug routes

---

# HTTP Methods

REST APIs commonly expose:

* GET
* POST
* PUT
* PATCH
* DELETE
* OPTIONS

Example:

```http id="nt1j5t"
OPTIONS /api/users HTTP/1.1
```

Method-specific behavior may expose:

* hidden functionality
* inconsistent authorization
* alternate processing logic
* unsupported methods

Attack Surface:

* method confusion
* missing method authorization
* unsupported HTTP methods
* hidden administrative methods

Method comparison targets:

```http id="bnjlwm"
GET /api/users/15
PUT /api/users/15
PATCH /api/users/15
DELETE /api/users/15
```

Observe:

* authorization differences
* response differences
* validation differences
* status code differences

---

# Object References

REST APIs commonly expose:

* numeric IDs
* UUIDs
* slugs
* internal object identifiers

Examples:

```http id="yjlwmx"
GET /api/users/15
GET /api/orders/1002
GET /api/invoices/INV-8821
```

Backend mapping commonly occurs directly into ORM queries.

Attack Surface:

* BOLA
* object enumeration
* tenant isolation failures
* relationship traversal

Testing targets:

* object ownership
* cross-account access
* sequential identifiers
* predictable UUID generation

---

# Query Parameters

Examples:

```http id="4a40lc"
GET /api/users?id=15&role=admin
```

Backend processing commonly includes:

* parameter extraction
* automatic type conversion
* ORM query generation
* filter construction

Attack Surface:

* parameter pollution
* hidden parameters
* authorization bypass
* filter manipulation
* query injection

Testing targets:

* duplicate parameters
* nested parameters
* unexpected parameters
* parameter type confusion

Examples:

```http id="pyo26n"
GET /api/users?id=1&id=2
```

```http id="yo7sdp"
GET /api/orders?is_admin=true
```

---

# JSON Request Bodies

Example:

```json id="6h1k94"
{
  "username": "alice",
  "role": "admin",
  "verified": true
}
```

Backend processing commonly includes:

* deserialization
* automatic object mapping
* ORM model hydration

Attack Surface:

* Mass Assignment
* hidden property injection
* nested object injection
* type confusion

Testing targets:

* hidden fields
* role fields
* internal flags
* nested objects
* relationship objects

Examples:

```json id="a9b9wd"
{
  "role": "admin"
}
```

```json id="vxy4jh"
{
  "is_verified": true
}
```

---

# Headers

Common headers:

```http id="dmy8ei"
Authorization
X-Forwarded-For
X-Original-URL
X-Rewrite-URL
X-User-ID
X-Internal-Request
```

Headers commonly influence:

* authentication
* authorization
* routing
* proxy behavior
* caching
* logging

Attack Surface:

* header trust abuse
* auth bypass
* route rewriting
* internal request spoofing

Testing targets:

* internal headers
* proxy headers
* forwarded identity headers
* IP-based trust logic

Examples:

```http id="r9u6ai"
X-Forwarded-For: 127.0.0.1
```

```http id="0mpsuz"
X-Original-URL: /admin
```

---

# Cookies

REST APIs may use:

* session cookies
* refresh tokens
* CSRF tokens

Example:

```http id="fzh3k1"
Cookie: session=abc123
```

Attack Surface:

* session fixation
* weak session validation
* insecure cookie configuration
* token replay

Testing targets:

* session reuse
* session invalidation
* cross-user session handling
* cookie scope behavior

---

# Authentication Endpoints

Examples:

```text id="zjlwm5"
/login
/register
/oauth/token
/refresh
/logout
```

Attack Surface:

* brute force
* token abuse
* refresh token abuse
* weak password reset logic
* MFA bypass

Testing targets:

* login rate limiting
* token lifecycle
* refresh token rotation
* password reset workflows
* OTP handling

---

# Authorization Boundaries

Authorization commonly enforced through:

* middleware
* controllers
* ORM filters
* service-layer validation

Attack Surface:

* BOLA
* Broken Function Level Authorization
* tenant isolation failures
* hidden admin functionality

Testing targets:

* cross-account access
* role differences
* privilege escalation
* method-specific authorization

Testing methodology:

* use multiple accounts
* compare accessible resources
* compare responses
* compare methods
* compare returned fields

---

# File Uploads

Common upload endpoints:

```text id="n5glg9"
/upload
/avatar
/documents
/import
```

Common content types:

```http id="qbq8b5"
multipart/form-data
```

Backend processing commonly includes:

* multipart parsing
* file storage
* MIME validation
* image processing
* antivirus scanning

Attack Surface:

* unrestricted uploads
* parser abuse
* path traversal
* content-type confusion
* image processing vulnerabilities

Testing targets:

* MIME types
* extensions
* polyglot files
* filename handling
* metadata processing

---

# Content-Type Processing

Common content types:

```http id="f4vkr5"
application/json
application/xml
multipart/form-data
application/x-www-form-urlencoded
```

Different parsers may trigger:

* different validation logic
* different deserialization logic
* different authorization paths

Attack Surface:

* parser inconsistencies
* validation bypass
* XML parser abuse
* content-type confusion

Testing methodology:

* replay same request with different content types
* compare:

  * validation behavior
  * status codes
  * response structure
  * authorization behavior

---

# API Versioning

Common structures:

```text id="k7m8hy"
/v1/
/v2/
/beta/
/internal/
```

Attack Surface:

* deprecated functionality
* hidden endpoints
* weaker authorization
* inconsistent validation

Testing targets:

* old versions
* beta routes
* undocumented versions
* internal endpoints

Compare:

* response fields
* authorization behavior
* validation behavior
* available functionality

---

# Administrative Functionality

Examples:

```text id="lbjlwm"
/admin
/manage
/internal
/debug
```

Exposure sources:

* frontend references
* JavaScript files
* mobile applications
* OpenAPI specs
* reverse proxy misconfiguration

Attack Surface:

* hidden admin routes
* internal APIs
* debug functionality
* privileged workflows

Testing targets:

* role validation
* hidden parameters
* method exposure
* internal-only endpoints

---

# OpenAPI / Swagger Exposure

Common locations:

```text id="e3ofm6"
/swagger
/swagger.json
/openapi.json
/api-docs
```

Exposed specifications may reveal:

* hidden endpoints
* parameters
* internal routes
* authentication methods
* schemas

Attack Surface:

* hidden functionality discovery
* undocumented methods
* admin APIs
* internal API structures

---

# API Gateways

Common gateway functions:

* routing
* authentication
* rate limiting
* request rewriting

Examples:

* Kong
* NGINX
* AWS API Gateway
* Apigee

Attack Surface:

* upstream header trust
* route rewriting
* internal path exposure
* gateway bypass

Testing targets:

* X-Original-URL
* X-Rewrite-URL
* forwarded headers
* internal routes

---

# Reverse Proxies

Common reverse proxies:

* NGINX
* HAProxy
* Envoy

Attack Surface:

* HTTP desynchronization
* cache poisoning
* internal route exposure
* header trust abuse

Testing targets:

* duplicate headers
* conflicting headers
* malformed requests
* chunked encoding behavior

---

# Caching Layers

Caching locations:

* CDN
* reverse proxy
* API gateway
* application cache

Attack Surface:

* cache poisoning
* cache deception
* authenticated response caching
* cache key confusion

Testing targets:

* cache headers
* authorization state
* shared cache behavior
* cache invalidation

---

# Internal APIs

Examples:

```text id="qjlwm2"
/internal/users
/internal/debug
/internal/admin
```

Internal APIs commonly:

* trust upstream services
* trust internal headers
* skip authorization checks

Exposure sources:

* SSRF
* reverse proxy misconfiguration
* direct exposure
* gateway bypass

Attack Surface:

* service impersonation
* internal trust abuse
* administrative functionality
* internal token trust

---

# Error Handling

Verbose errors may expose:

* framework details
* stack traces
* ORM behavior
* internal routes
* query structures

Example:

```json id="hjlwm7"
{
  "error": "SQLAlchemy.orm.exc.NoResultFound"
}
```

Attack Surface:

* framework fingerprinting
* ORM fingerprinting
* internal path disclosure
* backend logic disclosure

Testing targets:

* malformed JSON
* invalid types
* unsupported methods
* invalid object IDs
* malformed headers

---

# Response Structures

Example:

```json id="lqjlwm"
{
  "id": 1,
  "role": "admin",
  "internal": true
}
```

Response analysis targets:

* hidden fields
* authorization flags
* internal metadata
* object relationships
* serializer leakage

Compare responses across:

* users
* roles
* methods
* API versions
* content types
