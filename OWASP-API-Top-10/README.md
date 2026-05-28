# OWASP API Security Top 10

OWASP API Security Top 10 is a list of the most critical vulnerabilities commonly found in APIs.

Unlike traditional web applications, APIs directly expose:

* backend functionality
* objects
* business logic
* authentication systems
* internal workflows

Modern APIs are commonly consumed by:

* web frontends
* mobile applications
* third-party services
* internal microservices

Because APIs exchange structured data directly with backend systems:

* authorization failures become extremely dangerous
* hidden functionality becomes exposed
* backend trust assumptions become attack surface

Most API vulnerabilities occur because backend:

* trusts client-controlled data
* improperly validates authorization
* exposes sensitive object structures
* improperly handles business logic
* exposes internal functionality through APIs

---

# API1 — Broken Object Level Authorization (BOLA)

## Overview

Broken Object Level Authorization occurs when backend fails to validate whether an authenticated user is allowed to access a specific object.

APIs commonly expose object identifiers directly in:

* URLs
* JSON bodies
* GraphQL queries
* request parameters

Examples:

```http
GET /api/users/124 HTTP/1.1
```

```http
GET /api/orders/55 HTTP/1.1
```

```http
GET /api/messages/90 HTTP/1.1
```

Backend may correctly verify:

* user is authenticated

but fail to verify:

* whether requested object belongs to that user

As a result:

* attackers can access unauthorized objects
* attackers can modify other users’ resources
* attackers can perform unauthorized actions

BOLA is one of the most common API vulnerabilities because APIs heavily rely on:

* direct object references
* database identifiers
* resource-oriented architectures

---

## How BOLA Happens Internally

Backend commonly performs database queries directly using user-controlled identifiers.

Example backend logic:

```sql
SELECT * FROM users WHERE id=124
```

Problem:

* backend trusts attacker-controlled object ID
* ownership validation missing

Correct logic should validate:

```text
Does authenticated user own object 124?
```

If ownership validation missing:

* backend returns unauthorized object

---

## Common Object References

Examples:

* numeric IDs
* UUIDs
* GUIDs
* usernames
* account numbers
* email addresses

Example UUID:

```text
550e8400-e29b-41d4-a716-446655440000
```

---

## Common Vulnerabilities

* IDOR
* Horizontal Privilege Escalation
* Unauthorized Object Access
* Insecure Direct Object Reference

---

## Common Targets

* User profiles
* Orders
* Messages
* Payment information
* Invoices
* Support tickets
* Files
* API keys

---

## Testing Methodology

### Step 1 — Identify Object References

Look for object identifiers in:

* URLs
* JSON bodies
* GraphQL queries
* request parameters

Example:

```http
GET /api/users/124 HTTP/1.1
```

---

### Step 2 — Modify Object References

Change identifiers to different values.

Example:

```http
GET /api/users/125 HTTP/1.1
```

Observe whether backend:

* returns another user's data
* leaks unauthorized objects
* changes response structure
* returns authorization errors

---

### Step 3 — Compare Authorization Behavior

Use multiple accounts:

* normal user
* privileged user
* administrator

Compare:

* accessible objects
* accessible endpoints
* returned fields
* allowed actions

Check whether backend correctly validates:

* ownership
* permissions
* object visibility

---

### Step 4 — Test Different HTTP Methods

Example:

```http
GET /api/orders/100 HTTP/1.1
```

Test:

* PUT
* PATCH
* DELETE

Check whether backend incorrectly allows:

* modification
* deletion
* unauthorized actions

---

## Common Impacts

* Sensitive data exposure
* Unauthorized actions
* Account takeover
* Data modification
* Financial impact

---

# API2 — Broken Authentication

## Overview

Broken Authentication occurs when authentication mechanisms are:

* weak
* improperly implemented
* improperly validated

Authentication systems verify:

* user identity
* session validity
* token authenticity

If authentication fails:

* attackers may impersonate users
* bypass login systems
* hijack sessions
* forge tokens
* access protected APIs

APIs commonly use:

* JWT
* session cookies
* OAuth
* API keys
* access tokens
* refresh tokens

---

## How Authentication Works Internally

Authentication systems typically:

1. receive credentials
2. validate identity
3. generate session or token
4. associate authenticated state with user

Example:

```http
POST /api/login HTTP/1.1
```

Successful authentication commonly returns:

```http
Authorization: Bearer TOKEN
```

or:

```http
Set-Cookie: sessionId=abc123
```

Backend uses:

* session ID
* JWT
* token

to identify authenticated user in future requests.

---

## Common Vulnerabilities

* Weak JWT secrets
* JWT algorithm confusion
* Missing signature validation
* Session fixation
* Predictable tokens
* Token leakage
* Weak password reset
* OAuth misconfiguration

---

## JWT Authentication

JWT structure:

```text
HEADER.PAYLOAD.SIGNATURE
```

Example:

```text
eyJhbGciOiJIUzI1NiJ9...
```

Payload commonly contains:

* user ID
* roles
* expiration
* permissions

Backend validates:

* signature
* expiration
* claims

If validation weak:

* attackers may forge tokens
* attackers may escalate privileges

---

## Weak JWT Secrets

HMAC-based JWTs use shared secret for signing.

Weak secrets may allow attackers to:

* brute-force signing key
* forge valid tokens

Example weak secrets:

```text
secret
password
jwtsecret
```

---

## JWT Algorithm Confusion

Backend may incorrectly trust attacker-controlled `alg` value.

Example:

```json
{
  "alg": "none"
}
```

or:

```json
{
  "alg": "HS256"
}
```

Improper validation may allow:

* signature bypass
* forged tokens

---

## Testing Methodology

### Step 1 — Identify Authentication Mechanism

Check:

* Authorization headers
* session cookies
* OAuth redirects
* API keys

Examples:

```http
Authorization: Bearer TOKEN
```

```http
Cookie: sessionId=abc123
```

---

### Step 2 — Analyze Tokens

Decode JWTs and inspect:

* algorithms
* claims
* expiration
* roles
* user identifiers

Check for:

* weak algorithms
* exposed secrets
* insecure claims

---

### Step 3 — Test Token Validation

Test:

* modified payloads
* expired tokens
* unsigned tokens
* reused tokens

Observe whether backend:

* validates signatures
* validates expiration
* validates claims

---

### Step 4 — Analyze Session Handling

Check:

* session rotation
* logout invalidation
* predictable sessions
* session reuse

---

## Common Impacts

* Account takeover
* Authentication bypass
* Privilege escalation
* Unauthorized access

---

# API3 — Broken Object Property Level Authorization

## Overview

Broken Object Property Level Authorization occurs when backend improperly exposes or processes object properties.

Modern APIs commonly exchange JSON objects directly with backend systems.

Example:

```json
{
  "id": 1,
  "username": "alice",
  "role": "user"
}
```

Backend may:

* expose hidden fields
* expose internal properties
* allow modification of restricted properties

This commonly occurs because backend frameworks automatically map:

* JSON fields
* request parameters

into backend objects.

---

## How It Happens Internally

Backend frameworks often deserialize JSON directly into backend models.

Example request:

```json
{
  "isAdmin": true
}
```

If backend does not explicitly restrict properties:

* attacker-controlled fields may update sensitive backend attributes

This commonly affects:

* roles
* permissions
* billing flags
* internal configuration fields

---

## Common Vulnerabilities

* Mass Assignment
* Excessive Data Exposure
* Hidden Property Manipulation

---

## Excessive Data Exposure

Backend may return full backend objects instead of filtered responses.

Example:

```json
{
  "id": 1,
  "email": "alice@example.com",
  "passwordHash": "...",
  "internalRole": "admin"
}
```

Sensitive fields exposed because:

* backend trusts frontend filtering
* backend returns unnecessary properties

---

## Mass Assignment

Mass Assignment occurs when backend automatically maps user-controlled fields into backend objects.

Example:

```json
{
  "role": "admin"
}
```

If backend does not restrict allowed properties:

* attacker may modify sensitive fields

---

## Testing Methodology

### Step 1 — Analyze API Responses

Look for:

* hidden properties
* role fields
* internal flags
* backend metadata
* sensitive attributes

---

### Step 2 — Add Hidden Parameters

Add:

* role fields
* permission fields
* admin flags
* internal properties

Example:

```json
{
  "role": "admin"
}
```

Observe whether backend:

* accepts field
* updates object
* changes privileges

---

### Step 3 — Compare Different Roles

Use:

* normal user account
* administrator account

Compare:

* returned JSON fields
* hidden attributes
* exposed properties

Check whether low-privileged users receive:

* unnecessary data
* sensitive properties

---

## Common Impacts

* Privilege escalation
* Sensitive data exposure
* Role manipulation
* Internal information disclosure

---

# API4 — Unrestricted Resource Consumption

## Overview

Unrestricted Resource Consumption occurs when backend fails to properly restrict:

* request volume
* processing complexity
* resource usage

APIs may allow attackers to consume excessive:

* CPU
* memory
* bandwidth
* database resources
* application threads

This may lead to:

* denial of service
* infrastructure exhaustion
* degraded performance
* business logic abuse

---

## How It Happens Internally

Backend processes:

* every request
* every query
* every uploaded file

If backend lacks:

* rate limiting
* concurrency controls
* request validation
* resource restrictions

attackers may overload backend resources.

---

## Common Vulnerabilities

* Missing rate limiting
* GraphQL query abuse
* File upload abuse
* Infinite pagination
* Resource exhaustion
* Race conditions

---

## Missing Rate Limiting

Rate limiting controls:

* number of requests clients can send

Without rate limiting:

* brute force becomes possible
* OTP abuse becomes possible
* flooding attacks become possible

Example vulnerable endpoint:

```http
POST /api/login HTTP/1.1
```

---

## GraphQL Query Abuse

GraphQL APIs may process:

* deeply nested queries
* recursive queries
* expensive object traversal

Example:

```graphql
query {
  users {
    posts {
      comments {
        replies {
          author {
            posts {
              comments {
                replies {
                  id
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Large nested queries may:

* exhaust backend resources
* overload databases
* cause denial of service

---

## Testing Methodology

### Step 1 — Test Request Volume

Send repeated requests rapidly.

Observe:

* throttling
* blocking
* HTTP 429 responses
* response delays

---

### Step 2 — Test Concurrent Requests

Use:

* Turbo Intruder
* parallel requests

Check for:

* race conditions
* duplicate processing
* inconsistent backend state

---

### Step 3 — Test Resource-Heavy Inputs

Examples:

* large JSON bodies
* huge file uploads
* nested GraphQL queries

Observe:

* memory usage
* response times
* server instability

---

## Common Impacts

* Denial of Service
* Infrastructure exhaustion
* Business logic abuse
* Financial impact

---

# API5 — Broken Function Level Authorization

## Overview

Broken Function Level Authorization occurs when backend fails to properly restrict access to sensitive functionality.

Frontend applications commonly hide:

* admin panels
* privileged actions
* internal functionality

but backend APIs may still expose these functions directly.

Problem occurs when backend validates:

* authentication

but fails to validate:

* whether authenticated user can access specific function

---

## How It Happens Internally

Backend exposes:

* administrative APIs
* internal management endpoints
* privileged actions

Example:

```http
DELETE /api/users/1 HTTP/1.1
```

Backend executes request because:

* endpoint reachable
* authorization validation missing

---

## Common Vulnerabilities

* Privilege escalation
* Unauthorized admin access
* Function-level authorization bypass

---

## Common Targets

* Admin APIs
* User management APIs
* Payment management APIs
* Internal APIs
* Configuration endpoints

---

## Hidden Administrative APIs

Frontend applications often hide:

* admin buttons
* privileged functionality

but APIs remain accessible directly.

Examples:

```text
/api/admin
/api/internal
/api/config
```

Attackers may directly access:

* hidden endpoints
* undocumented methods
* privileged actions

---

## Testing Methodology

### Step 1 — Identify Sensitive Functions

Look for:

* admin routes
* internal APIs
* management functionality
* undocumented endpoints

---

### Step 2 — Test Low-Privilege Access

Use:

* normal user account

Attempt access to:

* admin endpoints
* privileged functions
* internal APIs

Observe whether backend:

* blocks request
* leaks data
* executes action

---

### Step 3 — Test Different HTTP Methods

Example:

```http
GET /api/users/1 HTTP/1.1
```

Test:

* PUT
* PATCH
* DELETE

Check whether backend exposes:

* modification functionality
* deletion functionality
* administrative actions

---

## Common Impacts

* Privilege escalation
* Unauthorized admin access
* Sensitive functionality exposure

---

# API6 — Unrestricted Access to Sensitive Business Flows

## Overview

Unrestricted Access to Sensitive Business Flows occurs when APIs fail to properly protect:
- sensitive workflows
- high-value actions
- abuse-prone business operations

In many cases:
- authentication works correctly
- authorization works correctly
- requests are technically valid

Problem exists in:
- workflow logic
- business rule enforcement
- abuse prevention mechanisms

APIs commonly expose business functionality directly through endpoints.

Examples:
- OTP verification
- checkout systems
- coupon redemption
- referral systems
- wallet transfers
- password reset flows

If backend does not properly enforce:
- workflow order
- action limits
- transaction restrictions
- anti-automation controls

attackers may abuse legitimate functionality.

---

## How It Happens Internally

Backend validates:
- request syntax
- authentication
- permissions

but fails to validate:
- workflow state
- action sequence
- business restrictions
- transaction limits

Example workflow:

```text
Request OTP → Verify OTP → Reset Password
```

If backend allows:
- unlimited OTP attempts
- replayed requests
- skipped steps

workflow becomes exploitable.

---

## Common Vulnerabilities

- OTP brute force
- Coupon abuse
- Referral abuse
- Race conditions
- Replay attacks
- Workflow bypass
- Payment manipulation
- Registration abuse

---

## OTP Brute Force

OTP verification APIs may fail to enforce:
- rate limiting
- attempt limits
- expiration validation

Example:

```http
POST /api/verify-otp HTTP/1.1
```

Weak protections may allow attackers to:
- brute-force OTPs
- bypass MFA
- hijack accounts

---

## Workflow Bypass

Backend may expect multi-step process but fail to validate sequence correctly.

Example workflow:

```text
/cart → /checkout → /payment → /confirm
```

Test direct access to:

```http
POST /api/payment/confirm HTTP/1.1
```

without completing previous workflow stages.

If backend validates only:
- authentication

but not:
- workflow state

payment confirmation may become accessible directly.

---

## Race Conditions

Sensitive actions may execute multiple times simultaneously before backend state updates complete.

Common targets:
- coupon redemption
- wallet transfers
- balance updates
- reward systems

Concurrent requests may cause:
- duplicate transactions
- repeated rewards
- inconsistent balances

---

## Testing Methodology

### Step 1 — Map Workflow

Identify:
- request sequence
- state transitions
- validation points
- tokens
- backend checks

Observe:
- IDs
- workflow states
- temporary tokens
- transaction references

---

### Step 2 — Test Step Skipping

Attempt:
- direct endpoint access
- skipping intermediate requests
- replaying old requests

Check whether backend validates:
- workflow order
- transaction state
- previous actions

---

### Step 3 — Test Automation Resistance

Check:
- rate limiting
- CAPTCHA validation
- replay protection
- OTP expiration
- request throttling

Attempt:
- brute force
- repeated requests
- rapid automation

---

### Step 4 — Test Concurrent Requests

Use:
- Turbo Intruder
- Burp Repeater
- parallel requests

Check for:
- duplicate processing
- inconsistent state changes
- repeated rewards
- balance corruption

---

## Common Impacts

- Financial abuse
- Account takeover
- Coupon exploitation
- Payment manipulation
- Reward abuse

---

# API7 — Server Side Request Forgery (SSRF)

## Overview

Server Side Request Forgery occurs when backend fetches attacker-controlled URLs or external resources.

Instead of client directly making request:
- backend server performs request internally

If attacker controls request destination:
- backend may access internal systems
- backend may access cloud metadata
- backend may access internal APIs
- backend may interact with restricted services

---

## How It Happens Internally

Backend receives URL input.

Example:

```json
{
  "url": "https://example.com/image.png"
}
```

Backend performs:

```text
Server → External Request → Response
```

If validation weak:
- attacker controls backend request target

Backend request originates from:
- internal infrastructure
- trusted network environment

This often bypasses:
- firewalls
- IP restrictions
- internal network segmentation

---

## Common SSRF Sources

- URL import functionality
- Webhook systems
- PDF generators
- Image fetchers
- Open Graph fetchers
- RSS readers
- Avatar importers

---

## Common Targets

### Localhost

```text
127.0.0.1
localhost
```

---

### Cloud Metadata Services

AWS:

```text
169.254.169.254
```

---

### Internal APIs

Examples:

```text
http://internal-api/
http://admin.internal/
```

---

## Protocol Testing

Test supported protocols:

```text
http://
https://
file://
gopher://
ftp://
```

Different protocols may expose:
- internal files
- internal services
- request smuggling opportunities

---

## Blind SSRF

Some backends:
- perform request
- do not return response

Use:
- Burp Collaborator
- external listener infrastructure

Check for:
- DNS interaction
- HTTP interaction

---

## Testing Methodology

### Step 1 — Identify URL Inputs

Look for:
- import features
- image URLs
- webhooks
- external integrations
- file fetchers

Example:

```json
{
  "url": "https://example.com"
}
```

---

### Step 2 — Test Internal Targets

Examples:

```text
127.0.0.1
localhost
169.254.169.254
```

Observe:
- response differences
- timeout behavior
- internal error messages
- metadata exposure

---

### Step 3 — Test Protocols

Examples:

```text
file://
gopher://
ftp://
```

Check whether backend:
- accepts protocols
- blocks protocols
- processes requests differently

---

### Step 4 — Test Blind SSRF

Use:
- Burp Collaborator
- external interaction server

Observe:
- DNS requests
- HTTP requests
- connection attempts

---

## Common Impacts

- Internal network access
- Cloud credential theft
- Internal service exposure
- Remote code execution

---

# API8 — Security Misconfiguration

## Overview

Security Misconfiguration occurs when:
- API infrastructure
- backend services
- security settings
- cloud environments

are improperly configured.

Modern APIs commonly depend on:
- reverse proxies
- API gateways
- cloud infrastructure
- caching layers
- container environments

Misconfigurations may expose:
- internal functionality
- sensitive data
- administrative interfaces
- insecure backend behavior

---

## Common Vulnerabilities

- CORS misconfiguration
- Verbose error messages
- Debug endpoints exposed
- HTTP request smuggling
- Host header injection
- Missing security headers
- Default credentials
- Insecure cloud configuration

---

## CORS Misconfiguration

CORS controls whether browsers allow:
- cross-origin API access

Dangerous configuration:

```http
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
```

If backend allows credentials with arbitrary origins:
- authenticated cross-origin requests may become possible

This may expose:
- API responses
- authenticated actions
- sensitive user data

---

## Verbose Error Messages

Backend errors may expose:
- stack traces
- framework names
- SQL queries
- internal paths
- backend technologies

Example:

```json
{
  "error": "SQL syntax error near ..."
}
```

Verbose errors improve attacker understanding of:
- backend structure
- parsing behavior
- infrastructure

---

## Debug Endpoints

Development or debugging interfaces may remain exposed in production.

Examples:

```text
/debug
/actuator
/swagger
/graphql
```

These endpoints may expose:
- internal APIs
- configuration data
- environment variables
- administrative functionality

---

## Testing Methodology

### Step 1 — Analyze Response Headers

Check:
- CORS headers
- security headers
- cache headers
- proxy headers

---

### Step 2 — Analyze Error Messages

Look for:
- stack traces
- SQL errors
- framework details
- internal paths
- backend technologies

---

### Step 3 — Test Infrastructure Behavior

Check:
- Host header handling
- proxy behavior
- request parsing
- cache behavior
- HTTP desynchronization

---

### Step 4 — Discover Administrative Interfaces

Search for:
- debug endpoints
- management APIs
- documentation portals
- internal dashboards

---

## Common Impacts

- Information disclosure
- Authentication bypass
- Internal application exposure
- Cache poisoning
- Infrastructure compromise

---

# API9 — Improper Inventory Management

## Overview

Improper Inventory Management occurs when organizations fail to properly manage:
- API versions
- deprecated APIs
- undocumented APIs
- shadow APIs
- internal APIs

Large API environments commonly expose:
- outdated endpoints
- forgotten APIs
- test environments
- legacy functionality

These APIs often:
- lack security fixes
- bypass modern protections
- expose hidden attack surface

---

## How It Happens Internally

Organizations continuously:
- deploy new API versions
- migrate infrastructure
- deprecate endpoints
- expose temporary APIs

Old APIs may remain accessible because:
- clients still depend on them
- routing still enabled
- infrastructure cleanup incomplete

Result:
- undocumented attack surface remains exposed

---

## Common Vulnerabilities

- Deprecated API exposure
- Shadow APIs
- Legacy authentication
- Debug APIs
- Unmaintained endpoints

---

## API Versioning

Common version patterns:

```text
/v1/
/v2/
/v3/
/beta/
/internal/
```

Older versions may:
- expose deprecated functionality
- miss authorization fixes
- use weaker authentication
- expose insecure business logic

---

## Shadow APIs

Shadow APIs are:
- undocumented
- unmanaged
- forgotten APIs

Often discovered through:
- JavaScript analysis
- mobile applications
- old documentation
- archived URLs
- endpoint fuzzing

---

## Testing Methodology

### Step 1 — Discover API Versions

Check for:

```text
/v1/
/v2/
/beta/
```

Compare:
- authentication behavior
- authorization controls
- response structures
- exposed functionality

---

### Step 2 — Analyze Documentation

Check:
- Swagger
- OpenAPI
- GraphQL schema

Look for:
- deprecated routes
- hidden APIs
- internal endpoints

---

### Step 3 — Discover Hidden APIs

Use:
- JavaScript analysis
- endpoint fuzzing
- archive sources
- mobile traffic analysis

---

### Step 4 — Compare Security Controls

Check whether older APIs:
- lack authentication
- expose sensitive data
- bypass rate limits
- expose admin functionality

---

## Common Impacts

- Unauthorized access
- Hidden attack surface
- Legacy vulnerability exposure
- Authentication bypass

---

# API10 — Unsafe Consumption of APIs

## Overview

Unsafe Consumption of APIs occurs when applications trust:
- external APIs
- third-party services
- internal microservices
- webhook providers

without properly validating:
- responses
- integrity
- authenticity
- trust boundaries

Modern applications heavily depend on:
- external integrations
- distributed services
- API-to-API communication

If backend blindly trusts external systems:
- malicious data may influence application logic
- trust boundaries may collapse
- backend systems may become compromised

---

## How It Happens Internally

Application receives data from:
- external APIs
- internal services
- webhooks
- partner systems

Backend assumes:
- source trusted
- response valid
- data integrity guaranteed

Example:

```text
Payment Provider → Backend → Account Credited
```

If backend blindly trusts response:
- attacker-controlled data may manipulate application state

---

## Common Vulnerabilities

- Unsafe deserialization
- Blind trust of third-party APIs
- SSRF via integrations
- Trust boundary failures
- Unvalidated external input
- Supply chain issues

---

## Webhook Trust Issues

Applications commonly process:
- payment notifications
- external callbacks
- webhook events

If webhook signatures not validated:
- attackers may forge requests

Example:

```json
{
  "payment_status": "success"
}
```

If backend trusts payload directly:
- fake payments may become accepted

---

## Unsafe Deserialization

Backend may deserialize:
- external objects
- serialized data
- complex structures

Unsafe deserialization may lead to:
- object injection
- remote code execution
- backend compromise

---

## Testing Methodology

### Step 1 — Identify External Integrations

Look for:
- webhook endpoints
- OAuth integrations
- payment APIs
- third-party services
- internal microservices

---

### Step 2 — Test Trust Assumptions

Check whether backend validates:
- signatures
- source authenticity
- response integrity
- trusted origins

---

### Step 3 — Analyze Data Processing

Check:
- deserialization behavior
- backend parsing
- object mapping
- trust boundaries

---

### Step 4 — Manipulate External Data

Modify:
- webhook payloads
- callback parameters
- integration responses

Observe whether backend:
- blindly trusts external data
- validates integrity correctly

---

## Common Impacts

- Remote code execution
- Payment manipulation
- Internal compromise
- Authentication bypass
- Data manipulation

---