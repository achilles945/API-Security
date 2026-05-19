# API-Sec-Playbook

Advanced API Pentesting Methodology, Vulnerability Playbooks, and Practical Exploitation Notes.

Focused on:

* API Recon
* BOLA / IDOR
* Authentication & JWT
* Mass Assignment
* GraphQL
* WebSockets
* Business Logic
* SSRF
* Race Conditions
* Modern API Attack Surface

Built as a practical operational playbook for real-world API security testing.

---

## 1. API Recon & Attack Surface Expansion

### Objective

Expand a single API into its full attack surface and identify all reachable endpoints, parameters, hidden functionality, and backend behavior.

---

### Phase 1 — API Discovery

* Identify API endpoints

* Common paths:

  * `/api/`
  * `/v1/`
  * `/v2/`
  * `/graphql`
  * `/swagger`
  * `/openapi.json`

* Identify:

  * REST APIs
  * GraphQL endpoints
  * WebSocket APIs
  * Mobile APIs
  * Internal APIs

---

### Phase 2 — Endpoint Mapping

* Capture traffic (Burp / mobile proxy)

* Extract:

  * Endpoints
  * Parameters
  * Methods
  * Headers
  * Tokens

* Identify:

  * Hidden endpoints
  * Deprecated APIs
  * Admin functionality
  * Internal functionality

---

### Phase 3 — JavaScript Analysis

* Collect JavaScript files

* Look for:

  * API endpoints
  * Hidden routes
  * Tokens
  * API keys
  * Internal URLs
  * GraphQL queries
  * WebSocket connections

* Extract:

  * Parameters
  * Function names
  * Request structure
  * Hidden logic

---

### Phase 4 — Parameter Discovery

* Identify all inputs:

  * Query parameters
  * JSON parameters
  * Headers
  * Cookies
  * Multipart data

* Find:

  * Hidden parameters
  * Unused parameters
  * Nested objects
  * Array parameters
  * Mass assignment opportunities

* Test:

  * Parameter pollution
  * Type confusion
  * JSON manipulation

---

### Phase 5 — Authentication Surface Mapping

* Identify:

  * Login APIs
  * Registration APIs
  * Password reset APIs
  * OTP / MFA APIs
  * OAuth flows

* Map:

  * JWT handling
  * Session handling
  * Refresh tokens
  * API keys
  * Authorization flow

---

### Phase 6 — Authorization Surface Mapping

* Identify:

  * User roles
  * Admin functionality
  * Object identifiers
  * Multi-tenant functionality

* Check:

  * Access controls
  * Resource ownership
  * Role restrictions

---

### Phase 7 — File & SSRF Surface

* Identify:

  * File upload APIs
  * File retrieval APIs
  * URL-fetching functionality
  * Import/export features

* Check:

  * File handling behavior
  * Internal requests
  * Storage behavior

---

### Phase 8 — Data Flow Mapping

For every input, identify where it goes:

* Database → SQL Injection
* Internal requests → SSRF
* File system → Path Traversal
* Object mapping → Mass Assignment
* Backend processing → Command Injection
* Serialization → Deserialization

---

### Phase 9 — API Behavior Analysis

* Identify:

  * Frameworks
  * API gateways
  * Reverse proxies
  * Rate limiting
  * CDN/WAF presence

* Observe:

  * Response patterns
  * Error messages
  * Authentication behavior
  * Header differences

---

### Phase 10 — Hidden Attack Surface

* Check:

  * Deprecated endpoints
  * Internal APIs
  * Debug functionality
  * Admin endpoints

* Look for:

  * Swagger exposure
  * GraphQL introspection
  * Hidden methods
  * Test functionality

---

### Output

* Endpoints & methods
* Parameters (all types)
* Authentication flows
* Authorization mapping
* Hidden functionality
* Request/response structure
* Internal behavior mapping

---

## 2. Input Surface Mapping

### Objective

Identify all controllable API inputs.

### Methodology

**Identify Inputs**

* Query parameters
* JSON data
* Headers
* Cookies
* Multipart data

**Trace Data Flow**

* Database interaction → SQL Injection
* Internal requests → SSRF
* Object mapping → Mass Assignment
* Serialization → Deserialization

**Micro-Tactics**

* Inject test values
* Compare responses
* Observe response structure
* Identify validation behavior

**API Awareness**

* Nested JSON objects
* Hidden fields
* Array parameters
* GraphQL queries
* WebSocket messages

---

## 3. Input-Based Vulnerabilities

### 3.1 SQL Injection

**Methodology**

* Inject payloads
* Compare responses
* Test boolean / time-based behavior

**Detection**

* Errors
* Response differences
* Time delays

---

### 3.2 NoSQL Injection

**Methodology**

* Inject JSON operators
* Manipulate backend queries

**Detection**

* Authentication bypass
* Response changes
* Backend errors

---

### 3.3 Mass Assignment

**Methodology**

* Add hidden JSON fields
* Modify object properties

**Detection**

* Privilege escalation
* Unauthorized property changes

---

### 3.4 SSRF

**Methodology**

* Identify URL-fetching functionality
* Target internal resources

---

### 3.5 File Upload

**Methodology**

* Upload malicious files
* Test execution / parsing behavior

---

### 3.6 Path Traversal

**Methodology**

* Manipulate file paths
* Access sensitive files

---

## 4. Authentication Testing

### Objective

Break API authentication mechanisms.

### Methodology

**JWT Testing**

* Weak signatures
* Missing validation
* Algorithm confusion

**Session Testing**

* Token reuse
* Token leakage
* Weak expiration

**OAuth Testing**

* Redirect URI issues
* Token leakage
* Scope abuse

**API Key Testing**

* Hardcoded keys
* Privilege abuse

---

## 5. Authorization Testing

### Objective

Access unauthorized data or functionality.

### Methodology

**BOLA / IDOR**

* Modify resource identifiers

**Broken Function Level Authorization**

* Access restricted endpoints

**Vertical Escalation**

* Access admin functionality

**Horizontal Escalation**

* Access other user data

---

## 6. Server-Side Vulnerabilities

### SSRF

* Test internal endpoints
* Test cloud metadata access
* Test internal APIs

### Deserialization

* Test serialized objects
* Test object manipulation

### XXE

* Inject XML entities
* Read files / trigger SSRF

---

## 7. Business Logic Testing

### Objective

Break API workflows and application logic.

### Methodology

* Skip workflow steps
* Manipulate request sequences
* Abuse payment flows
* Repeat actions
* Test race conditions
* Abuse trust relationships

---

## 8. Vulnerability Chaining

### Objective

Combine multiple API weaknesses.

### Examples

* BOLA → account modification → account takeover
* SSRF → internal API access → privilege escalation
* Mass assignment → admin role → full compromise
* JWT weakness → authentication bypass

---

## Core Mental Model

1. Expand API attack surface
2. Map endpoints and parameters
3. Understand backend behavior
4. Test authorization aggressively
5. Analyze request processing
6. Chain vulnerabilities
