---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance"
date:   2026-01-09 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [jwt,authentication-bypass,none-algorithm,broken-authentication]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Finance
><br/><b>Vulnerabilities Covered:</b>
<br/>
JWT None Algorithm Attack
<br/>
Broken Authentication
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a **JWT (JSON Web Token) authentication bypass** vulnerability caused by improper algorithm validation. The application accepts JWTs with the `alg` header set to `none`, which instructs the server to skip signature verification entirely. By modifying the JWT header to use the none algorithm and removing the signature portion, an attacker can tamper with the payload claims specifically changing the `role` from `user` to `admin`, without needing to know the secret key used for signing. This allows complete authentication bypass and privilege escalation to administrative access, enabling retrieval of sensitive data from protected admin endpoints.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis and Endpoint Discovery

Register a new account on the Shady Oaks Financial trading platform. After successful registration, use JS Recon Buddy to enumerate all application endpoints and identify potential targets. Notice the `/api/admin/flag` endpoint, which indicates administrative functionality that requires elevated privileges to access.

![Endpoints](/images/bug-forge/daily/shady-oaks-financial/jwt/js-recon-buddy-endpoints.png)

---

### Step 2 - JWT Token Inspection

Using Caido's JWT Analyzer plugin, examine the structure of the authentication token received after login. Send any authenticated request to the plugin for analysis.

![JWT Analyzer - Dashboard](/images/bug-forge/daily/shady-oaks-financial/jwt/jwt-analyser-dashboard.png)

Click on JWT Editor to inspect and manipulate the token structure.

![JWT Editor](/images/bug-forge/daily/shady-oaks-financial/jwt/jwt-editor.png)

The decoded JWT payload reveals critical claims including `id`, `username`, and `role`. The current user has the role set to `user`, which restricts access to administrative endpoints.

---

### Step 3 - None Algorithm Attack

Exploit the JWT vulnerability by modifying the token header to use the `none` algorithm. In the JWT Analyzer, change the `alg` field from its current value to `none`, and update the payload claims to escalate privileges. Set the `role` to `admin`, update the `id` to `1`, and adjust other identifying fields as needed.

![JWT Manipulation](/images/bug-forge/daily/shady-oaks-financial/jwt/jwt-token-manipulation.png)

---

### Step 4 - Token Validation

Test the crafted JWT by sending a request to a protected endpoint such as `/api/verify-token` with the modified token in the `Authorization` header. If the server accepts the token and returns a successful response instead of an authentication error, the none algorithm vulnerability is confirmed. This proves the server does not properly validate the signature algorithm and accepts unsigned tokens.

![Token Validation](/images/bug-forge/daily/shady-oaks-financial/jwt/token-validation.png)

---

### Step 5 - Flag Retrieval

With a validated admin JWT token, access the `/api/admin/flag` endpoint identified during reconnaissance. The server processes the request using the forged token's claims, granting administrative access and returning the flag in the response.

![Flag](/images/bug-forge/daily/shady-oaks-financial/jwt/flag.png)

---

### Impact
- Complete authentication bypass through algorithm manipulation
- Unauthorized privilege escalation from user to administrator
- Access to sensitive administrative functionality and data
- Ability to impersonate any user by modifying JWT claims
- Full compromise of application authorization controls

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** JWT None Algorithm Attack / Broken Authentication
- **Attack Surface:** Authentication token handling
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity

---

### Root Cause

The backend JWT library is configured to accept the `none` algorithm, which instructs the server to skip signature verification entirely. When the server receives a token with `alg: none`, it processes the payload claims without validating the signature, allowing attackers to forge tokens with arbitrary claims.

This vulnerability typically occurs when:
- JWT libraries have the none algorithm enabled by default
- Explicit algorithm validation is not enforced during token verification
- The server trusts the `alg` claim from the token header rather than using a server-side whitelist

---

### Remediation

**Algorithm Enforcement:**
- Explicitly specify allowed algorithms on the server side during token verification
- Never trust the `alg` claim from the token header
- Disable or reject the `none` algorithm in all JWT library configurations

**Example (Node.js with jsonwebtoken):**
```javascript
jwt.verify(token, secretKey, { algorithms: ['HS256'] });
```

**Additional Controls:**
- Use asymmetric algorithms (RS256, ES256) for enhanced security where key distribution is feasible
- Implement token expiration (`exp` claim) to limit the window for token abuse
- Store sensitive authorization data server-side rather than in JWT claims
- Implement token revocation mechanisms for compromised tokens
- Add comprehensive logging for authentication failures and anomalous token patterns
- Regularly audit JWT library configurations and update dependencies
- Consider using a well-tested authentication framework that handles JWT security by default

---
