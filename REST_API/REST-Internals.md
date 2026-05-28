# REST Internals

---

# What Is REST

REST (Representational State Transfer) is an HTTP-based API architecture using:

* resources
* URLs
* HTTP methods
* stateless request handling

REST APIs expose backend functionality through resource-oriented routes.

Examples:

```text id="j5g7m8"
/users
/orders
/products
/payments
/messages
```

Resources commonly map internally to:

* database entities
* ORM models
* backend service objects

Example:

```http id="ltogtr"
GET /api/users/15 HTTP/1.1
```

Backend interpretation:

```text id="xpcqak"
Retrieve user object with ID 15
```

---

# REST Architecture

REST APIs commonly follow:

```text id="3e43g4"
Client
→ API Gateway
→ Reverse Proxy
→ Router
→ Controller
→ Service Layer
→ ORM
→ Database
```

Common backend components:

* routers
* controllers
* middleware
* serializers
* ORM layers
* authentication services
* caching layers

Common implementations:

* Express.js
* Django REST Framework
* Spring Boot
* FastAPI
* ASP.NET Web API

---

# Resource-Oriented Routing

Routes commonly represent:

* objects
* collections
* workflows

Examples:

```text id="1e4oqn"
/users/15
/orders/91
/products/7
```

Common route patterns:

```text id="1q8dgv"
GET /users/15
POST /orders
PATCH /users/15
DELETE /products/5
```

Backend route mapping:

```python id="9q6h6g"
@app.route("/users/<id>")
def get_user(id):
```

Framework behavior:

* parameter extraction
* automatic route matching
* request parsing
* request object creation

Attack Surface:

* object references
* parameter injection
* route normalization inconsistencies
* hidden routes
* undocumented methods

---

# Stateless Request Processing

REST APIs commonly operate statelessly.

Authentication state stored client-side.

Each request contains:

* authentication context
* authorization context
* request data

Example:

```http id="q6e6oc"
Authorization: Bearer TOKEN
```

Backend behavior:

```text id="g0bg7r"
Request
→ Token Validation
→ User Context Injection
→ Authorization Checks
```

Attack Surface:

* token replay
* weak token validation
* internal token trust
* forwarded authentication abuse

---

# HTTP Request Structure

REST APIs use HTTP requests containing:

* request line
* headers
* cookies
* query parameters
* request body

Example:

```http id="v37vzt"
POST /api/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "product_id": 5,
  "quantity": 1
}
```

Request components processed independently by backend layers.

---

# HTTP Method Processing

| Method | Backend Operation |
| ------ | ----------------- |
| GET    | Read resource     |
| POST   | Create resource   |
| PUT    | Replace resource  |
| PATCH  | Modify resource   |
| DELETE | Remove resource   |

Example:

```http id="9l2v0j"
DELETE /api/users/5 HTTP/1.1
```

Backend operation:

```text id="4nls89"
Delete user object 5
```

Different methods may trigger:

* different middleware
* different authorization logic
* different validation behavior
* different serializers

Attack Surface:

* method confusion
* missing method authorization
* unsupported method exposure
* hidden HTTP methods

---

# Request Processing Lifecycle

Processing Flow:

```text id="86v31x"
Client
→ CDN
→ Reverse Proxy
→ API Gateway
→ Authentication Middleware
→ Authorization Middleware
→ Router
→ Controller
→ Service Layer
→ ORM
→ Database
→ Serializer
→ Response
```

Processing stages introduce:

* trust boundaries
* parsing boundaries
* serialization behavior
* routing behavior
* authorization boundaries

---

# Authentication Processing

Common mechanisms:

* sessions
* JWT
* OAuth
* API keys

JWT Processing Flow:

```text id="4js2bd"
Client Login
→ Credential Validation
→ JWT Generation
→ Client Storage
→ Request Validation
```

Example:

```http id="1gwk1w"
Authorization: Bearer eyJhbGciOi...
```

Authentication middleware commonly:

* extracts token
* validates signature
* validates expiration
* loads user object
* injects identity context

Internal Flow:

```text id="az9ecw"
Request
→ JWT Middleware
→ User Context Injection
→ Authorization Logic
```

Common Failure Patterns:

* missing signature validation
* weak JWT secrets
* algorithm confusion
* upstream header trust
* internal token trust

---

# Authorization Processing

Authorization controls:

* object access
* action execution
* endpoint access
* method access
* field visibility

Authorization locations:

* middleware
* controllers
* service layer
* ORM queries

Insecure pattern:

```python id="2j9ytr"
user = User.query.get(user_id)
return jsonify(user)
```

Secure pattern:

```python id="24x5y0"
user = User.query.filter_by(
    id=user_id,
    owner=current_user.id
).first()
```

Attack Surface:

* BOLA
* Broken Function Level Authorization
* tenant isolation failures
* hidden admin functionality
* method-level authorization gaps

---

# Middleware Processing

Processing Flow:

```text id="nq8r3d"
Request
→ Logging Middleware
→ Authentication Middleware
→ Authorization Middleware
→ Validation Middleware
→ Controller
```

Middleware behavior:

* request rewriting
* header injection
* identity propagation
* validation enforcement
* authorization enforcement

Common Failure Patterns:

* incorrect middleware order
* middleware bypass
* internal route exposure
* reverse proxy bypass
* upstream header trust

---

# Query Parameters

Example:

```http id="0v9mx1"
GET /api/users?id=15&role=admin
```

Backend behavior:

* parameter extraction
* automatic type conversion
* ORM query generation

Attack Surface:

* parameter pollution
* hidden parameters
* query manipulation
* authorization bypass

---

# JSON Processing

Example:

```json id="v4vl7d"
{
  "username": "alice",
  "role": "admin"
}
```

Processing Flow:

```text id="p8i1l8"
JSON Body
→ JSON Parser
→ Deserializer
→ Backend Object
→ ORM Model
→ Database
```

Attack Surface:

* Mass Assignment
* hidden fields
* nested object injection
* parser inconsistencies
* type confusion

---

# Serialization & Deserialization

## Deserialization

Request data converted into backend objects.

Example:

```json id="lz4xxo"
{
  "email": "alice@example.com"
}
```

Backend mapping:

```python id="vv0yrf"
User(email="alice@example.com")
```

Attack Surface:

* hidden property injection
* Mass Assignment
* nested object injection

---

## Serialization

Backend objects converted into API responses.

Backend object:

```python id="9bjlwm"
User(
    id=1,
    email="alice@example.com",
    password_hash="..."
)
```

Serialized response:

```json id="pr0t43"
{
  "id": 1,
  "email": "alice@example.com",
  "password_hash": "..."
}
```

Attack Surface:

* excessive data exposure
* hidden field leakage
* internal metadata exposure
* serializer overexposure

---

# ORM Interaction

Common ORMs:

* SQLAlchemy
* Hibernate
* Sequelize
* Django ORM

Processing Flow:

```text id="z1bcl4"
Request
→ ORM Query
→ Database
→ ORM Object
→ Serializer
→ JSON Response
```

Insecure pattern:

```python id="8jlwmw"
User.query.get(id)
```

without ownership validation.

Attack Surface:

* BOLA
* object enumeration
* relationship exposure
* hidden object traversal

---

# Content-Type Processing

Common content types:

```http id="v3vdxg"
application/json
application/xml
multipart/form-data
application/x-www-form-urlencoded
```

Different parsers may:

* validate differently
* deserialize differently
* enforce security differently

Attack Surface:

* parser inconsistencies
* content-type confusion
* upload abuse
* XML parser abuse
* validation bypass

---

# API Gateways

Gateway functions:

* authentication
* routing
* rate limiting
* request rewriting
* logging

Processing Flow:

```text id="3hmm4e"
Client
→ API Gateway
→ Backend API
```

Examples:

* Kong
* AWS API Gateway
* NGINX
* Apigee

Gateway trust assumptions:

* forwarded identity headers
* upstream authentication state
* internal network trust

Attack Surface:

* internal header trust
* route exposure
* auth bypass
* upstream spoofing

---

# Reverse Proxies

Common functions:

* TLS termination
* caching
* load balancing
* request forwarding

Examples:

* NGINX
* HAProxy
* Envoy

Attack Surface:

* header trust abuse
* HTTP desynchronization
* cache poisoning
* internal route exposure

---

# REST Caching

Caching layers:

* CDN
* reverse proxy
* API gateway
* application cache

Attack Surface:

* authenticated response caching
* cache key confusion
* cache poisoning
* authorization bypass through cache

---

# API Versioning

Common version structures:

```text id="trjn6g"
/v1/
/v2/
/beta/
/internal/
```

Older versions may:

* expose deprecated functionality
* use weaker authorization
* expose hidden endpoints
* bypass security controls

---

# Microservices

Common architecture:

```text id="r2itd6"
Frontend API
├── Auth Service
├── User Service
├── Payment Service
```

Internal services commonly communicate through APIs.

Internal APIs frequently:

* trust upstream services
* trust internal headers
* skip authorization checks

Attack Surface:

* SSRF pivoting
* internal API abuse
* service impersonation
* internal token trust

---

# Common Backend Failure Patterns

## Trusting Client-Controlled IDs

Example:

```http id="9zg4gn"
GET /api/users/5
```

without ownership validation.

---

## Returning Full ORM Objects

Example:

```python id="i8p4ny"
return jsonify(user.__dict__)
```

May expose:

* internal fields
* password hashes
* authorization flags
* hidden metadata

---

## Missing Method Authorization

Example:

```http id="l8jkto"
GET /users/5
```

protected correctly while:

```http id="t3gb5u"
DELETE /users/5
```

lacks authorization enforcement.

---

## Trusting Upstream Headers

Examples:

```http id="olokd7"
X-User-ID
X-Forwarded-For
X-Internal-Request
```

Improper validation may create:

* auth bypass
* internal trust abuse
* privilege escalation

---

## Hidden Administrative Routes

Examples:

```text id="3b3ahj"
/admin
/internal
/debug
/manage
```

Exposure sources:

* forgotten routes
* reverse proxy misconfiguration
* legacy deployments
* internal tooling
