---
layout: post
title:  "BugForge - Daily - Copy Pasta"
date:   2026-02-11 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication,session-hijacking]
categories: [BugForge,daily,copy-pasta]
---


# Daily - Copy Pasta
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Authentication
<br/>
Session Hijacking via Predictable Session Tokens
<br/>
<br/>
<b>Summary:</b>
<br/>
The CopyPasta application uses a predictable session token scheme where session identifiers are derived by computing the MD5 hash of the username and then Base64-encoding the result. After registering a new account and intercepting the `/api/snippets` request, the `session` value was decoded from Base64 to reveal an MD5 hash that matched the MD5 of the registered username. By computing the MD5 hash of the `admin` username, Base64-encoding it, and replaying the `/api/snippets` request with the forged session token, the application returned a response containing the challenge flag in the `X-flag` header, demonstrating that an attacker can impersonate any user without credentials by simply knowing or guessing their username.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Create a new account and reference the first endpoint we hit after logging in `/api/snippets` notice the `session` value.

![Request with sessionId](/images/bug-forge/daily/copy-pasta/session-id/session-id.png)

By the looks of it the session is Base64 encoded we'll use [Cyber Chef](https://gchq.github.io/CyberChef) to decode it

![Base64 Decoded](/images/bug-forge/daily/copy-pasta/session-id/base64-decoded.png) The decoded value looks like an MD5 hash, we'll try hasing our username `Zwarts` and compare the value.

![MD5 Hash](/images/bug-forge/daily/copy-pasta/session-id/md5-hash.png)

Next we'll get the session id for the `admin` user

![Admin session](/images/bug-forge/daily/copy-pasta/session-id/admin-session.png)

We'll replay the `/api/snippets` request and change the session id to the admin session id, notice the flag in the `X-flag` header

![Flag](/images/bug-forge/daily/copy-pasta/session-id/flag.png)

---

### Impact
- Complete account impersonation of any user by forging session tokens from publicly known or guessable usernames
- Unauthorized access to sensitive data and functionality belonging to privileged accounts such as administrators
- Full bypass of authentication controls without requiring valid credentials
- Potential for mass account takeover as all user sessions can be deterministically computed

---

### Vulnerability Classification
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Broken Authentication / Predictable Session Tokens
- **Attack Surface:** Session management via `/api/snippets` endpoint
- **CWE:** CWE-330 - Use of Insufficiently Random Values

---

### Root Cause
The application generates session tokens by computing the MD5 hash of the username and Base64-encoding the result, making session identifiers entirely deterministic and predictable. Because session values are derived solely from a known input (the username) without any secret, randomness, or server-side entropy, any attacker who knows or can guess a target username can forge a valid session token and authenticate as that user without credentials.

---

### Remediation
- Generate session tokens using a cryptographically secure random number generator (CSPRNG) with sufficient entropy
- Ensure session identifiers are not derived from or correlated with any user-controllable or predictable input
- Implement server-side session validation that binds tokens to authenticated sessions rather than trusting client-supplied values
- Enforce session expiration and rotation policies to limit the window of token reuse
- Use established session management frameworks or libraries that follow security best practices
- Monitor for anomalous session activity such as multiple concurrent sessions or sudden privilege changes

---
