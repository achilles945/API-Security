# Core API Fundamentals

## API Fundamentals

- What APIs are
- Why APIs exist
- How APIs work internally
- Request-response model
- API architectures
- REST APIs
- GraphQL APIs
- SOAP APIs
- WebSockets
- gRPC
- API gateways
- API versioning
- API documentation

## HTTP & APIs

- HTTP protocol
- HTTP methods
- Request structure
- Response structure
- Headers
- Status codes
- Cookies
- Sessions
- Statelessness
- CORS

## API Data Processing

- JSON
- XML
- Form-urlencoded
- Multipart/form-data
- Serialization
- Deserialization
- Base64 encoding

## Authentication & Authorization

- Authentication
- Authorization
- Sessions
- Tokens
- JWT
- OAuth
- API keys
- RBAC
- ABAC

## API Infrastructure

- Reverse proxies
- API gateways
- CDNs
- Load balancers
- Microservices
- Service-to-service communication
- Rate limiting
- API caching

---

# API Fundamentals

---

## What Is an API

API (Application Programming Interface) is a communication layer that allows different systems to exchange data and functionality.

In modern applications:
- frontend does not directly access database
- mobile applications do not directly communicate with backend logic
- services communicate through APIs

Instead:

```text
Client → API → Backend Logic → Database
```

API acts as an interface between:
- client
- backend application
- databases
- internal services

The API receives:
- requests
- data
- authentication information

Then:
- processes logic
- interacts with backend systems
- returns response

---

## Why APIs Exist

APIs exist because modern applications are distributed systems.

Different components:
- frontend
- mobile apps
- databases
- payment providers
- authentication services

must communicate with each other.

Without APIs:
- frontend would directly access database
- systems would become tightly coupled
- scaling infrastructure becomes difficult

APIs solve this by:
- separating frontend and backend
- standardizing communication
- allowing independent scaling
- enabling multiple clients

Example:

```text
Web App
Mobile App
Desktop App
```

can all communicate with:
- same backend API

---

## API Communication Model

Most APIs use request-response communication.

Flow:

```text
Client → API → Backend Services → Response
```

Typical flow:
1. Client sends request
2. API validates request
3. Authentication occurs
4. Authorization occurs
5. Backend logic executes
6. Database interaction occurs
7. Response returned

---

## Client vs Server

### Client

Client initiates communication.

Examples:
- browser
- mobile app
- frontend JavaScript
- third-party applications

Client:
- sends requests
- processes responses
- displays data

---

### Server

Server processes requests and returns responses.

Examples:
- API server
- backend application
- authentication server

Server:
- validates requests
- executes business logic
- interacts with databases
- returns responses

---

# API Architectures

Different API architectures are designed to solve different communication problems.

Because of this:
- request structure changes
- backend processing changes
- attack surface changes
- pentesting methodology changes

Understanding architecture is critical because:
- vulnerabilities depend heavily on architecture
- backend behavior differs significantly
- request parsing differs significantly

---

## REST APIs

REST (Representational State Transfer) is the most common API architecture used in modern applications.

REST exposes application functionality using:
- URLs
- HTTP methods
- JSON responses

REST is heavily built around HTTP protocol behavior.

---

### How REST APIs Work

REST organizes application data into resources.

Examples:

```text
/users
/orders
/products
/messages
```

Each resource represents:
- object
- collection
- backend functionality

REST APIs use HTTP methods to define actions.

Example:

```http
GET /users/1 HTTP/1.1
```

Backend interpretation:

```text
Retrieve user with ID 1
```

Example:

```http
DELETE /users/1 HTTP/1.1
```

Backend interpretation:

```text
Delete user with ID 1
```

Same endpoint behaves differently depending on:
- HTTP method
- parameters
- authentication state

---

### Why REST Uses URLs

REST uses URLs because HTTP itself is resource-oriented.

Example:

```text
/users/1
```

naturally maps to:
- user object with ID 1

This design makes:
- routing simple
- frontend integration easy
- APIs scalable

But it also creates attack surface around:
- object identifiers
- endpoint access
- HTTP methods

---

### Why REST Usually Uses JSON

REST APIs commonly use JSON because:
- lightweight
- easy to parse
- language-independent
- frontend-friendly

Example:

```json
{
  "id": 1,
  "name": "alice"
}
```

Frontend JavaScript can directly process JSON objects.

Compared to XML:
- JSON has smaller payload size
- lower parsing overhead
- simpler structure

---

### REST Characteristics

REST APIs are commonly:
- stateless
- resource-based
- URL-driven
- JSON-based

Stateless means:
- server does not automatically remember previous requests
- every request must contain authentication and required context

Benefits:
- scalability
- easier load balancing
- distributed infrastructure

---

### REST Pentesting Perspective

REST APIs commonly expose:
- object identifiers
- internal functionality
- hidden endpoints
- admin operations

Because REST heavily relies on object IDs:
- authorization testing becomes critical

Most common REST vulnerabilities occur because:
- backend trusts object identifiers
- backend exposes hidden functionality
- backend accepts unexpected JSON fields

---

## GraphQL APIs

GraphQL is a query language for APIs.

Unlike REST:
- client controls what fields are returned
- backend exposes functionality through a flexible query system

Most GraphQL APIs use:

```text
/graphql
```

---

### Why GraphQL Exists

REST APIs often cause:
- over-fetching
- under-fetching
- multiple requests

Example:

Frontend only needs:
- username

but REST may return:

```json
{
  "username": "alice",
  "email": "alice@example.com",
  "address": "...",
  "phone": "..."
}
```

GraphQL solves this by allowing client to request:
- exact fields required

---

### How GraphQL Works

Client sends query specifying:
- exactly what data should be returned

Example:

```graphql
query {
  user(id: 1) {
    name
    email
  }
}
```

Server processes query and returns only requested fields.

---

### GraphQL Characteristics

- Single endpoint architecture
- Flexible queries
- Nested object traversal
- Client-controlled responses

One endpoint may expose:
- users
- payments
- admin functionality
- internal objects

This creates very large attack surface.

---

### GraphQL Pentesting Perspective

GraphQL commonly exposes:
- hidden fields
- internal objects
- nested relationships
- schema information

Attackers commonly test:
- introspection
- field-level authorization
- nested query abuse
- excessive data exposure

---

## SOAP APIs

SOAP (Simple Object Access Protocol) uses XML-based communication.

SOAP is common in:
- enterprise systems
- banking systems
- legacy infrastructure

SOAP requests use XML.

Example:

```xml
<soap:Envelope>
  <soap:Body>
    <GetUser>
      <id>1</id>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

---

### Why SOAP Uses XML

SOAP was designed before modern JSON-based APIs.

XML provides:
- strict structure
- schema validation
- complex document support

SOAP systems heavily rely on:
- XML parsers
- XML schemas
- WSDL files

---

### WSDL Files

WSDL describes:
- methods
- operations
- parameters

Example:

```text
/service?wsdl
```

WSDL helps clients understand:
- how API communicates
- what methods exist
- what data is required

But WSDL may also expose:
- hidden functionality
- internal operations
- admin methods

---

### SOAP Pentesting Perspective

SOAP heavily relies on XML parsers.

Because of this:
- parser behavior becomes critical
- XML vulnerabilities become important

Common SOAP vulnerabilities:
- XXE
- XML Injection
- Information disclosure

---

## WebSockets

WebSockets allow persistent two-way communication between:
- client
- server

Unlike HTTP:
- connection remains open
- communication becomes continuous

Example:

```text
Client ↔ Server
```

---

### Why WebSockets Exist

Traditional HTTP requires:
- repeated requests

This creates:
- latency
- unnecessary overhead
- inefficient real-time communication

WebSockets solve this by:
- maintaining persistent connection
- allowing continuous message exchange

Used in:
- chat applications
- live dashboards
- trading systems
- multiplayer games

---

### WebSocket Workflow

1. Client sends upgrade request
2. Server upgrades connection
3. Persistent communication begins

Example:

```http
GET /chat HTTP/1.1
Upgrade: websocket
Connection: Upgrade
```

After upgrade:
- both sides continuously exchange messages

---

### WebSocket Pentesting Perspective

WebSockets commonly expose:
- hidden actions
- internal commands
- live message handling

Attackers test:
- message tampering
- unauthorized subscriptions
- hidden actions
- authentication flaws

---

## gRPC

gRPC is a high-performance API framework designed for:
- internal services
- microservices
- backend communication

Uses:
- HTTP/2
- Protocol Buffers

---

### Why gRPC Exists

Traditional APIs:
- use text-based formats like JSON/XML
- create larger payloads
- require more parsing overhead

gRPC improves performance by:
- using binary serialization
- reducing payload size
- reducing processing overhead

This makes gRPC:
- faster
- more efficient
- better for internal infrastructure

---

### Protocol Buffers

gRPC uses Protocol Buffers instead of JSON.

Protocol Buffers:
- serialize data into compact binary format

Example JSON:

```json
{
  "id": 1,
  "name": "alice"
}
```

contains:
- field names
- formatting characters
- larger payload

Binary serialization removes unnecessary overhead.

Result:
- faster communication
- smaller requests
- lower bandwidth usage

---

### gRPC Pentesting Perspective

gRPC commonly exists internally.

Developers often trust:
- internal services
- service-to-service communication

This may create:
- weak authorization
- hidden functionality
- exposed internal methods

---

# HTTP & APIs

HTTP is foundation of most APIs.

Understanding HTTP is critical because APIs heavily rely on:
- requests
- responses
- headers
- methods

Most API vulnerabilities occur because:
- backend misinterprets requests
- authentication fails
- authorization fails
- headers are trusted incorrectly

---

## HTTP Request Structure

Example:

```http
POST /api/login HTTP/1.1
Host: api.example.com
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "username": "admin",
  "password": "password"
}
```

Request contains:
- request line
- headers
- body

---

## HTTP Methods

Methods define:
- action backend should perform

| Method | Purpose |
| --- | --- |
| GET | Retrieve data |
| POST | Create data |
| PUT | Replace resource |
| PATCH | Modify resource |
| DELETE | Remove resource |

Different methods often trigger:
- different backend functionality
- different authorization checks

---

## Headers

Headers provide metadata about request.

Examples:

```http
Authorization
Content-Type
Origin
Cookie
```

Headers commonly affect:
- authentication
- authorization
- routing
- caching

Many vulnerabilities occur because backend trusts headers incorrectly.

---

## Status Codes

Status codes indicate request result.

| Code | Meaning |
| --- | --- |
| 200 | Success |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 500 | Server Error |

Status codes help testers understand:
- backend behavior
- authorization logic
- hidden functionality

---

# Authentication & Authorization

---

## Authentication

Authentication verifies identity.

Question answered:

```text
Who are you?
```

Examples:
- passwords
- sessions
- JWT
- OAuth

---

## Authorization

Authorization verifies permissions.

Question answered:

```text
What are you allowed to access?
```

Most critical API vulnerabilities are authorization-related.

---

## Sessions

Session-based authentication stores session state on server.

After login:
- server creates session
- browser stores session ID in cookie

Example:

```http
Cookie: sessionId=abc123
```

Server uses session ID to identify user.

---

## Tokens

Token-based authentication stores authentication inside token itself.

Example:

```http
Authorization: Bearer TOKEN
```

Backend validates token for every request.

This approach is commonly used in:
- APIs
- mobile applications
- distributed systems

---

## JWT

JWT (JSON Web Token) is signed token format.

Structure:

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

Common vulnerabilities:
- weak secrets
- missing signature validation
- algorithm confusion

---

# API Infrastructure

---

## Reverse Proxies

Reverse proxies sit between:
- clients
- backend APIs

Example:

```text
Client → Reverse Proxy → API Server
```

Functions:
- routing
- TLS termination
- filtering
- caching

Examples:
- Nginx
- HAProxy

---

## API Gateways

API gateways manage API traffic.

Functions:
- authentication
- rate limiting
- logging
- request routing

Examples:
- Kong
- AWS API Gateway

---

## Microservices

Modern applications commonly use microservices.

Instead of:
- one large backend

application becomes:
- many small services

Example:

```text
Frontend API
 ├── Auth Service
 ├── Payment Service
 ├── User Service
```

Services communicate internally using APIs.

---

## Service-to-Service Communication

Internal services communicate with each other through APIs.

Example:

```text
Auth Service → User Service
```

These internal APIs are often:
- less protected
- highly trusted

This creates important internal attack surface.

---

# Core 

API security failures usually occur because:
- backend trusts user input
- authorization checks fail
- hidden functionality is exposed
- internal services trust each other incorrectly
- backend interprets requests unexpectedly