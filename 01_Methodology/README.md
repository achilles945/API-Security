# API METHODOLOGY

---

## 0. RULES

* Focus on high-impact API functionality
* Do not fuzz blindly without understanding API logic
* If no progress in 20 minutes, pivot
* Always ask: can this lead to account takeover, privilege escalation, sensitive data exposure, or remote code execution

---

## 1. TARGET TRIAGE

### Identify high-value API functionality first

* Authentication APIs
* Registration APIs
* Password reset APIs
* Account management APIs
* Payment APIs
* File upload APIs
* Admin APIs
* Mobile APIs
* Internal APIs

---

### Immediate attack indicators

* `user_id`, `account_id`, `profile_id` → test for BOLA / IDOR
* JWT / API tokens → test validation weaknesses
* File upload endpoints → test for execution
* URL parameters → test for SSRF
* Bulk actions → test mass assignment
* Hidden parameters → test privilege escalation
* GraphQL → test introspection and excessive data exposure

---

## 2. API ATTACK SURFACE EXPANSION

### API Discovery

Identify:

* REST APIs
* GraphQL endpoints
* WebSocket APIs
* Mobile APIs
* Internal APIs
* Versioned APIs

Common paths:

```text
/api/
/v1/
/v2/
/graphql
/swagger
/openapi.json
```

---

### JavaScript Analysis

Primary source for API discovery.

Extract:

* Endpoints
* Hidden routes
* Parameters
* Tokens
* API keys
* Internal functionality
* Unused API calls

---

### API Mapping

Identify:

* Endpoints
* Methods
* Parameters
* Authentication requirements
* Request/response structure
* User roles

Methods:

```http
GET
POST
PUT
PATCH
DELETE
OPTIONS
```

---

### Parameter Discovery

Identify all controllable input:

* JSON parameters
* Query parameters
* Headers
* Cookies
* Multipart data
* Nested objects
* Arrays

Look for:

* Hidden parameters
* Unused parameters
* Mass assignment opportunities
* Parameter pollution

---

### Documentation Discovery

Check for:

```text
/swagger
/swagger.json
/api-docs
/openapi.json
/graphql
```

These may expose:

* hidden endpoints
* internal functionality
* admin APIs
* undocumented parameters

---

## 3. INPUT → PROCESSING → IMPACT

### For every API input

1. Where does it go
2. What processes it
3. What data does it access
4. What functionality does it influence
5. Can it affect other users

---

### Data Flow Mapping

* Database → SQL Injection
* Backend requests → SSRF
* File system → Path Traversal
* System commands → Command Injection
* Object mapping → Mass Assignment
* Serialization → Deserialization

---

### Rule

If input reaches:

* sensitive functionality
* internal systems
* privileged actions
* backend processing

prioritize testing immediately.

---

## 4. ATTACK PRIORITY

### 1. Authorization

Most critical API vulnerability category.

Test:

* Access to other users' data
* Resource ID modification
* Direct object access
* Admin functionality access
* Hidden endpoints

Focus:

* BOLA / IDOR
* Broken Function Level Authorization
* Privilege escalation

---

### 2. Authentication

Test:

* JWT validation
* Token predictability
* Token leakage
* Weak session handling
* OAuth issues
* API key exposure

Goal:

* Account takeover

---

### 3. Business Logic

Test:

* Workflow bypass
* Step skipping
* Race conditions
* Duplicate actions
* Payment manipulation
* Rate limit bypass

---

### 4. Mass Assignment

Test whether hidden JSON fields can modify sensitive values.

Example:

```json
{
  "role": "admin"
}
```

Check for:

* Role modification
* Permission changes
* Internal flags
* Hidden fields

---

### 5. Sensitive Data Exposure

Check API responses for:

* Internal IDs
* Tokens
* Emails
* Phone numbers
* Debug information
* Hidden fields

---

### 6. Input-Based Vulnerabilities

* SQL Injection
* NoSQL Injection
* SSTI
* Command Injection
* SSRF
* XXE

---

## 5. API-SPECIFIC TESTING

### HTTP Method Testing

Test unsupported methods.

Example:

```http
PUT
PATCH
DELETE
OPTIONS
```

Check whether:

* access controls differ
* functionality changes
* validation weakens

---

### Content-Type Manipulation

Test:

```http
application/json
application/xml
multipart/form-data
text/plain
```

Check whether backend processes requests differently.

---

### Rate Limit Testing

Check:

* Login APIs
* OTP APIs
* Password reset APIs
* Verification endpoints

Test:

* Concurrent requests
* IP rotation
* Header manipulation

---

### GraphQL Testing

Check:

* Introspection enabled
* Excessive data exposure
* Nested query abuse
* Authorization flaws
* Field-level access control

---

### WebSocket Testing

Check:

* Authentication
* Authorization
* Message tampering
* Hidden actions
* Subscription abuse

---

## 6. EXPLOITATION PROCESS

### Step 1 — Confirm

Verify behavior is controllable.

---

### Step 2 — Analyze

Identify:

* Request structure
* Backend behavior
* Validation logic
* Authentication checks

---

### Step 3 — Bypass

Attempt:

* Parameter modification
* Method switching
* Encoding tricks
* JSON manipulation
* Header manipulation

---

### Step 4 — Escalate

Attempt:

* Access other users
* Gain admin access
* Access internal APIs
* Reach code execution

---

## 7. CHAINING

### Core Question

What can this API vulnerability lead to.

---

### Common Chains

* BOLA → account modification → account takeover
* SSRF → internal API access → privilege escalation
* Mass assignment → admin role → full compromise
* JWT weakness → authentication bypass
* Race condition → payment abuse

---

## 8. MODERN API ATTACK SURFACE

* BOLA
* Mass assignment
* JWT attacks
* GraphQL abuse
* Race conditions
* WebSocket vulnerabilities
* API desynchronization
* Server-side parameter pollution

---

## 9. TOOL USAGE

### Workflow

1. Capture API traffic
2. Map endpoints
3. Test manually
4. Fuzz selectively
5. Automate when needed

---

### Primary Tools

* Burp Suite Professional
* Postman
* Insomnia
* mitmproxy
* ffuf
* jq

---

### Burp Extensions

* Autorize
* JWT Editor
* Param Miner
* Content Type Converter
* Logger++

---

### Rule

Tools help discover attack surface.

Manual testing finds vulnerabilities.

---

## 10. EFFICIENCY

* Focus on sensitive endpoints first
* Prioritize authorization testing
* Avoid blind fuzzing
* Move quickly between attack paths
* APIs usually expose functionality directly — exploit logic flaws early

---

## 11. VERIFICATION

* Confirm response differences
* Verify real authorization impact
* Eliminate false positives
* Confirm exploitation consistency
* Verify whether impact affects other users

---
