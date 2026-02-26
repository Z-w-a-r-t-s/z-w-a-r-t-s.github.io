---
layout: post
title:  "BugForge - Daily - Sokudo"
date:   2026-01-15 20:40
image:  /images/bug-forge/bugforge-logo.png
tags:   [api-versioning,broken-authentication,idor,jwt-manipulation]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo
><br/><b>Vulnerabilities Covered:</b>
<br/>
API Versioning Vulnerability
Broken Authentication
IDOR (Insecure Direct Object Reference)
JWT Token Manipulation
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates how legacy API endpoints can introduce critical security vulnerabilities when not properly deprecated or secured. The application exposes both `/v1` and `/v2` API versions, where the newer `/v2` endpoints enforce proper JWT validation while the legacy `/v1` endpoints fail to verify token signatures and role claims. By discovering the `/v1/admin/flag` endpoint through enumeration and manipulating JWT claims (changing the role to "admin" and user ID to "1"), an attacker can bypass authentication and authorization controls to access administrative functionality that would otherwise be protected.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Initial Reconnaissance and API Version Discovery

![Register Request](/images/bug-forge/daily/Sokudo/jwt/registration-request.png)

After registering an account and logging in, all HTTP traffic was monitored using a proxy tool (Caido). Analysis of the requests revealed that all API endpoints used the `/v2` prefix (e.g., `/v2/login`, `/v2/stats`, `/v2/register`).

![API Versioning](/images/bug-forge/daily/Sokudo/jwt/api-versioning.png)

This immediately prompted investigation into whether older API versions existed. Testing the same endpoints with `/v1` instead of `/v2` confirmed that legacy endpoints were still accessible.

**Key Observation:** Applications that expose multiple API versions without proper deprecation or security parity create opportunities for attackers to bypass newer security controls by targeting older implementations.

---

### Step 2 - Endpoint Enumeration
JS Recon Buddy was used to enumerate available endpoints across the application. Reviewing the results, several `admin`-tagged routes stood out as interesting targets.

![JS Recon Buddy](/images/bug-forge/daily/Sokudo/jwt/endpoints-js-recon-buddy.png)

Both endpoints were tested immediately using the original tokens from the authenticated session.

---

### Step 3 - JWT Algorithm None Attack
With admin-tagged endpoints identified, the next step was to attempt a JWT Algorithm None attack. By setting the JWT algorithm to `"none"`, the signature requirement is dropped entirely. The token can be forged with arbitrary claims and the server is expected to accept it without verification.

The authenticated user's JWT was decoded to inspect its structure:

![JWT Editor](/images/bug-forge/daily/Sokudo/jwt/jwt-editor.png)

The following modifications were made to craft the forged token:
1. Set the `alg` header to `"none"` and removed the signature
2. Changed `role` from `"user"` to `"admin"`
3. Changed `user_id` to `1`, assuming the admin account holds the first ID

![JWT Manipulation](/images/bug-forge/daily/Sokudo/jwt/jwt-manipulation.png)

The forged token was tested against both the v2 endpoint (`/v2/admin/flag`) and the v1 endpoint (`/v1/admin/flag`). As expected, the v2 endpoint rejected the token, its signature verification was intact. The v1 endpoint, however, accepted the token without complaint:

![JWT None Algorithm Response](/images/bug-forge/daily/Sokudo/jwt/jwt-none-algorith-response.png)

With the forged token accepted, the v1 endpoint returned the flag:

![Flag](/images/bug-forge/daily/Sokudo/jwt/flag.png)

---


### Impact
- Complete bypass of authentication controls on legacy API endpoints
- Unauthorized access to administrative functionality
- Ability to impersonate any user by manipulating the user ID claim
- Potential for privilege escalation across the entire application
- Demonstrates how maintaining legacy endpoints without security parity creates critical vulnerabilities

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Broken Authentication, JWT Signature Bypass
- **Secondary Issue:** IDOR (Insecure Direct Object Reference)
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The application made several critical security mistakes:

1. **Inconsistent Security Controls:** The `/v1` API endpoints lacked the security improvements implemented in `/v2`, specifically JWT signature verification.

2. **Missing Token Signature Validation:** Legacy endpoints accepted JWT tokens without verifying the cryptographic signature, allowing attackers to forge claims.

3. **ID-Based Authorization:** The application relied solely on the `user_id` claim from the token for authorization decisions without additional verification, enabling IDOR attacks.

4. **Legacy Endpoint Exposure:** Outdated API versions remained accessible in production without proper deprecation or equivalent security controls.

---

### Remediation
- Implement consistent security controls across ALL API versions
- Properly deprecate and remove legacy API endpoints that cannot be secured
- Always verify JWT signatures using strong cryptographic algorithms (RS256, ES256)
- Never trust claims from tokens without signature verification
- Implement defense in depth by validating user permissions server-side against a database, not just token claims
- Use API versioning strategies that don't expose vulnerable legacy code
- Conduct security reviews when maintaining multiple API versions
- Implement monitoring and alerting for access to deprecated endpoints
- Consider using API gateways to enforce consistent authentication policies across all versions
- Regular security audits of all exposed endpoints, including legacy versions

---
