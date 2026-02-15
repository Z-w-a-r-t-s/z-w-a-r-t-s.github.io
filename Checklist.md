# CTF Vulnerability Checklist

Comprehensive checklist derived from all BugForge writeups. Use this as a systematic guide for any web CTF.

---

## 1. Reconnaissance & Information Gathering

- [ ] Register an account and analyze the registration request/response (user IDs, roles, tokens)
- [ ] Map all application features and endpoints via the UI
- [ ] Intercept all traffic in a proxy (Burp/Caido) - API calls may not be visible in browser URLs
- [ ] Inspect client-side JavaScript source code for hardcoded API endpoints, service identifiers, and secrets
- [ ] Run directory/endpoint enumeration with wordlists (e.g. `seclists/Discovery/Web-Content/common.txt`)
- [ ] Check for API versioning (v1 vs v2) - legacy endpoints may lack security controls
- [ ] Look for GraphQL endpoints (`/graphql`) and run introspection queries to enumerate the full schema
- [ ] Check response headers and error messages for information disclosure (stack traces, DB type, framework info)
- [ ] Analyze token formats (JWT, timestamps, MD5 hashes) - decode and understand their structure

---

## 2. Authentication & Session Management

- [ ] **Predictable Tokens** - Check if auth tokens are based on timestamps, usernames, or other guessable values (e.g. ISO 8601 `YYYYMMDDHHmmss`, `MD5(username)`)
- [ ] **JWT None Algorithm** - Change `alg` to `none` and remove the signature to bypass verification
- [ ] **JWT Claim Tampering** - Modify `role`, `user_id`, or `isAdmin` claims; re-sign if secret/key is available
- [ ] **JWT Secret Extraction** - If SQLi is found, check for JWT secrets stored in the database
- [ ] **JWT Key Confusion** - If RSA keys are exposed (via SSRF, LFI, or misconfiguration), forge tokens with `jwt_tool`
- [ ] **Information Disclosure enabling Session Hijacking** - Check if leaderboards, user lists, or public APIs expose timestamps/data that can reconstruct tokens
- [ ] **Brute Force Login** - Test for missing rate limiting on authentication endpoints
- [ ] **SQL Injection Auth Bypass** - Try `' OR 1=1--` and similar payloads on login forms

---

## 3. Access Control (IDOR & Privilege Escalation)

- [ ] **IDOR on GET endpoints** - Change numeric IDs in URLs (`/api/users/1`, `/api/stats/1`, `/api/products/1`) to access other users' data
- [ ] **IDOR on PUT/POST/DELETE endpoints** - Modify resource identifiers in update/delete requests to affect other users' resources
- [ ] **Horizontal Privilege Escalation** - Access another user's profile, stats, orders, or snippets by changing the user ID
- [ ] **Vertical Privilege Escalation** - Add/modify `role=admin` or `isAdmin=true` in registration or upgrade requests
- [ ] **Parameter Tampering** - Check if `role`, `user_id`, `organization_id` or similar fields are accepted from client input
- [ ] **Mass Assignment** - Send extra fields in registration/profile update requests (e.g. `user_role`, `is_admin`)
- [ ] **HTTP Verb Tampering** - If GET is blocked, try PUT, POST, PATCH, DELETE on the same endpoint
- [ ] **Missing Function-Level Access Control** - Try accessing `/admin/*` endpoints directly as a regular user
- [ ] **Multi-Tenant Isolation** - Test type confusion on tenant/org IDs (string vs integer, boolean vs string)

---

## 4. Injection Attacks

### SQL Injection
- [ ] **Detection** - Insert `'` in parameters; look for errors or changed behavior
- [ ] **Boolean-based** - Test `AND 1=1` vs `AND 1=2` for different responses
- [ ] **UNION-based** - Determine column count with `ORDER BY N`, then `UNION SELECT null,...`
- [ ] **SQLite specifics** - Use `sqlite_master` for table enumeration, `pragma_table_info('table')` for columns, `group_concat()` for multi-row extraction
- [ ] **INSERT/UPDATE injection** - Use `||` concatenation operator with subqueries when injection point is in write operations
- [ ] **Second-Order/Stored SQLi** - Malicious input stored in one place, executed when retrieved elsewhere (e.g. job title stored, executed when admin views applications)
- [ ] **Auth Bypass** - `' OR 1=1--` on login forms
- [ ] **Test all input points** - URL params, POST body, headers, cookies, GUIDs, search fields

### XSS (Cross-Site Scripting)
- [ ] **Stored XSS** - Inject `<script>` tags in user-generated content (profiles, comments, posts)
- [ ] **Check for dangerous rendering** - Look for `dangerouslySetInnerHTML` (React), `v-html` (Vue), `innerHTML` usage
- [ ] **OOB Data Exfiltration** - Use `fetch()` in XSS payloads to send stolen data (cookies, tokens) to attacker-controlled server

### Template Injection (SSTI)
- [ ] **Detection** - Inject `{{7*7}}` or `<%= 7*7 %>` and check if `49` is returned
- [ ] **EJS** - `<%= process.env %>` for environment variable disclosure, RCE payloads
- [ ] **Other engines** - Jinja2, Twig, Pug - use engine-specific payloads

### XXE / XInclude
- [ ] **Standard XXE** - Define external entity in DTD to read files (`file:///etc/passwd`)
- [ ] **XInclude** - When traditional XXE is blocked, use `<xi:include>` to include files
- [ ] **Check XML-accepting endpoints** - File uploads, SOAP endpoints, any XML body parsers

---

## 5. Server-Side Vulnerabilities

### SSRF (Server-Side Request Forgery)
- [ ] **Identify user-controllable URLs** - Look for parameters like `url`, `callback`, `redirect`, `webhook` that trigger server-side requests
- [ ] **Internal service discovery** - Fuzz for internal endpoints (`/auth`, `/admin`, `/config`, `/internal`)
- [ ] **Key/credential harvesting** - Look for exposed private keys, config files, or metadata endpoints on internal services
- [ ] **Cloud metadata** - Try `http://169.254.169.254/` for cloud instance metadata

### Prototype Pollution (Node.js/Express)
- [ ] **Detection** - Send `{"__proto__": {"isAdmin": true}}` in JSON body to merge/update endpoints
- [ ] **Common targets** - Object merge functions, deep copy utilities, settings/preferences endpoints
- [ ] **Exploitation** - Pollute `isAdmin`, `role`, `authorized` properties that are checked but never explicitly set

### Path Traversal / LFI
- [ ] **Directory traversal** - Use `../` sequences in file path parameters to access files outside intended directory
- [ ] **Bypass filters** - URL encoding (`%2e%2e%2f`), double encoding, null bytes

---

## 6. Business Logic Flaws

- [ ] **Price/Value Manipulation** - Intercept checkout/payment requests and modify prices, quantities, or totals to zero or negative values
- [ ] **Negative Value Injection** - Submit negative amounts for tips, points, refunds, or transfers
- [ ] **Insufficient Entropy** - Check if gift cards, vouchers, OTPs, or reset tokens use predictable/short codes that can be brute-forced
- [ ] **Missing Server-Side Validation** - Check if the server trusts client-supplied values for anything security-critical (prices, roles, permissions)
- [ ] **Race Conditions (TOCTOU)** - Send concurrent requests to exploit check-then-act patterns (balance checks, coupon redemption, inventory)
- [ ] **Race Conditions in Checkout** - Modify cart between validation and processing steps
- [ ] **Refund/Credit Abuse** - Test arbitrary refund amounts or repeated redemptions
- [ ] **Type Confusion** - Send arrays where strings expected, booleans where IDs expected, integers where strings expected

---

## 7. Security Bypass Techniques

- [ ] **WAF Bypass via Payload Inflation** - Pad payloads to ~8KB+ to exceed WAF inspection buffer while keeping the malicious portion intact
- [ ] **Filter Bypass** - If certain characters are blocked, try alternative encodings, case variations, or equivalent syntax
- [ ] **Missing CSRF Protection** - Check if state-changing requests (password change, email update, account deletion) lack CSRF tokens
- [ ] **API Gateway Type Confusion** - Send unexpected types (boolean `true`, arrays, objects) for ID parameters to discover hidden services

---

## 8. Information Disclosure

- [ ] **Error messages** - Database errors, stack traces, framework versions
- [ ] **API oversharing** - Endpoints returning more fields than the UI displays (passwords, internal IDs, timestamps)
- [ ] **GraphQL introspection** - Full schema including sensitive fields like `password`, `secret`, `token`
- [ ] **Source code exposure** - Client-side JS revealing API structure, internal service URLs, hardcoded keys
- [ ] **Environment variables** - Via SSTI (`process.env`) or misconfigured debug endpoints
- [ ] **Registration/login responses** - User IDs, role info, token patterns revealed

---

## 9. Methodology Reminders

- [ ] Always intercept and inspect every HTTP request/response in your proxy
- [ ] Test every parameter you find - URL path, query params, POST body, headers, cookies
- [ ] Check registration responses for user IDs and role assignments
- [ ] Monitor background API calls (SPAs make calls not visible in the URL bar)
- [ ] When you find one vulnerability, look for chaining opportunities (e.g. SSRF -> key leak -> JWT forgery)
- [ ] Try all HTTP methods on interesting endpoints
- [ ] Use sequential ID enumeration (0-100) with automation tools on every endpoint that uses IDs
- [ ] For SQLite: `sqlite_master`, `pragma_table_info`, `group_concat()`, `||` concatenation
- [ ] For JWT: always decode first, check algorithm, look for secret/key exposure
- [ ] Create multiple test accounts to verify cross-account access

---

## Tools Checklist

- [ ] **Proxy**: Burp Suite or Caido (intercept, repeat, automate)
- [ ] **JWT**: jwt_tool, jwt.io, Caido JWT Analyzer
- [ ] **Fuzzing**: SecLists wordlists, Burp Intruder, Caido Automate
- [ ] **GraphQL**: GraphQL Voyager, introspection queries
- [ ] **Encoding**: CyberChef
- [ ] **Recon**: JS Recon Buddy (endpoint discovery from JS files)
- [ ] **SSRF**: QuickSSRF (Caido plugin)
- [ ] **Race Conditions**: Burp Turbo Intruder, parallel request sending
