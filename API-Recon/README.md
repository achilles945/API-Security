# API Recon

## API Recon

- API documentation discovery
- Endpoint discovery
- HTTP method testing
- Content type testing
- Parameter discovery
- Hidden endpoint discovery
- JavaScript analysis
- API versioning
- Authentication analysis
- GraphQL discovery
- WebSocket discovery

---

# API Recon

API recon is the process of identifying:
- endpoints
- parameters
- request formats
- authentication mechanisms
- hidden functionality
- backend behavior

Good recon directly affects:
- attack surface visibility
- vulnerability discovery
- exploitation success

---

## API Documentation Discovery

API documentation often exposes:
- endpoints
- methods
- parameters
- request schemas
- authentication methods

Common locations:

```text
/swagger
/swagger-ui
/swagger/index.html
/openapi.json
/openapi.yaml
/api-docs
/redoc
```

---

## Swagger / OpenAPI

Swagger/OpenAPI files describe:
- API routes
- methods
- parameters
- request formats
- authentication requirements

Example:

```json
{
  "/users": {
    "get": {}
  }
}
```

Useful for:
- endpoint mapping
- hidden functionality discovery
- request generation

---

## Endpoint Discovery

Endpoints expose backend functionality.

Examples:

```text
/api/users
/api/orders
/api/payments
```

Endpoints can be discovered from:
- frontend traffic
- JavaScript files
- documentation
- mobile applications
- error messages

---

## JavaScript Analysis

JavaScript files often expose:
- hidden endpoints
- internal APIs
- parameter names
- tokens
- GraphQL queries

Example:

```javascript
fetch("/api/users")
axios.post("/api/login")
```

Also look for:
- admin routes
- staging APIs
- internal domains

---

## HTTP Method Testing

HTTP methods define backend actions.

| Method | Purpose |
| --- | --- |
| GET | Retrieve data |
| POST | Create data |
| PUT | Replace data |
| PATCH | Modify data |
| DELETE | Remove data |
| OPTIONS | Discover supported methods |

---

Different methods may expose:
- hidden functionality
- admin operations
- unauthorized actions

Example:

```http
GET /api/users/1
```

Test:

```http
DELETE /api/users/1
```

Check whether:
- authorization changes
- hidden actions exist

---

## OPTIONS Method

OPTIONS may reveal supported methods.

Example:

```http
OPTIONS /api/users HTTP/1.1
```

Response:

```http
Allow: GET, POST, PUT, DELETE
```

Useful for:
- endpoint mapping
- hidden method discovery

---

## Content-Type Testing

APIs process requests differently depending on content type.

Common types:

```http
application/json
application/xml
multipart/form-data
application/x-www-form-urlencoded
```

---

Different parsers may:
- validate input differently
- process input differently
- expose different vulnerabilities

Example:
- XML parser vulnerable to XXE
- JSON parser not vulnerable

---

## Testing Different Content Types

Original request:

```http
Content-Type: application/json
```

Test:

```http
Content-Type: application/xml
```

Observe:
- parsing behavior
- validation changes
- error messages

---

## Parameter Discovery

Parameters control backend behavior.

Parameters may exist in:
- query strings
- JSON bodies
- headers
- cookies
- path parameters

---

## Hidden Parameters

Backend may support undocumented parameters.

Example:

```json
{
  "username": "alice",
  "isAdmin": true
}
```

Frontend may never expose:
- `isAdmin`

but backend may still process it.

---

## Parameter Discovery Sources

Discover parameters from:
- JavaScript files
- API documentation
- error messages
- response objects
- GraphQL schema
- fuzzing

---

## Hidden Endpoint Discovery

Many APIs expose undocumented endpoints.

These often contain:
- admin functionality
- test functionality
- deprecated APIs

---

## Endpoint Fuzzing

Use wordlists to discover:
- hidden routes
- internal APIs
- admin endpoints

Examples:

```text
/admin
/internal
/debug
/test
/staging
```

---

## API Version Discovery

APIs commonly expose multiple versions.

Examples:

```text
/v1/
/v2/
/beta/
```

Older versions may:
- lack security fixes
- expose deprecated functionality

---

## Authentication Analysis

Identify:
- JWT usage
- session cookies
- API keys
- OAuth flows
- refresh tokens

---

## JWT Indicators

Example:

```http
Authorization: Bearer eyJhbGci...
```

JWT tokens contain:
- header
- payload
- signature

---

## Session Indicators

Example:

```http
Cookie: sessionId=abc123
```

---

## OAuth Indicators

Look for:

```text
/oauth
/authorize
/token
```

---

## GraphQL Discovery

Most GraphQL APIs use:

```text
/graphql
```

GraphQL may expose:
- schema
- hidden queries
- internal objects

---

## Introspection Testing

Test whether introspection enabled.

Example:

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

Introspection may expose:
- full backend schema
- hidden functionality

---

## WebSocket Discovery

WebSockets commonly used for:
- chat
- notifications
- live updates

---

## WebSocket Indicators

Look for:

```http
Upgrade: websocket
Connection: Upgrade
```

Or:

```text
ws://
wss://
```

---

## Error Message Analysis

Error messages may expose:
- framework details
- backend structure
- parameter names
- SQL errors
- stack traces

Always analyze:
- parser errors
- validation errors
- authorization errors

---

## Response Analysis

Compare responses carefully.

Look for:
- status code differences
- response length differences
- hidden fields
- authorization differences

Response analysis often reveals:
- hidden behavior
- backend logic
- vulnerable functionality

---

# API Recon Workflow

## Step 1 — Discover Documentation

Check:
- Swagger
- OpenAPI
- GraphQL

---

## Step 2 — Capture Traffic

Map:
- endpoints
- methods
- parameters

---

## Step 3 — Analyze JavaScript

Extract:
- hidden endpoints
- parameter names
- internal APIs

---

## Step 4 — Test Methods

Test:
- GET
- POST
- PUT
- PATCH
- DELETE
- OPTIONS

---

## Step 5 — Test Content Types

Switch between:
- JSON
- XML
- form-urlencoded

---

## Step 6 — Discover Parameters

Use:
- response analysis
- documentation
- fuzzing

---

## Step 7 — Discover Hidden Endpoints

Use:
- wordlists
- JS analysis
- version discovery

---

## Step 8 — Analyze Authentication

Identify:
- JWT
- OAuth
- sessions
- API keys