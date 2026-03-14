---
layout: post
title:  "BugForge - Daily - Gift Lab"
date:   2026-03-13 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication,brute-force]
categories: [BugForge,daily,gift-lab]
---

# Daily - Gift Lab
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Authentication - Brute Force<br/>
<br/>
<b>Summary:</b>
<br/>
The Gift Lab application contains a **`Broken Authentication`** flaw where the `adminAccessToken` cookie issued at login has a predictable structure - only the last 3 characters vary across accounts. By generating a wordlist of all possible suffixes and brute forcing the `/administrator` endpoint, a valid admin token can be identified without credentials.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Reconnaissance

After registering a new account and logging in, the application redirects and returns two cookies: a `token` and an `adminAccessToken`. The presence of an admin-specific token signals there is a protected admin endpoint worth targeting.

![Fuzzing](/images/bug-forge/daily/gift-lab/administrator-bruteforce/fuzzing.png)

After fuzzing with `seclists/Discovery/Web-Content/common.txt` we find that an `/administrator` path exists.

![Admin Page](/images/bug-forge/daily/gift-lab/administrator-bruteforce/admin-path.png)

Visiting the admin page returns an access denied error:

![Admin access denied](/images/bug-forge/daily/gift-lab/administrator-bruteforce/admin-access-denied.png)

### Step 2 - Identifying Token Predictability

Inspecting the `adminAccessToken` across multiple newly registered accounts reveals a pattern - only the last 3 characters of the token change between accounts:

![Admin Token 1](/images/bug-forge/daily/gift-lab/administrator-bruteforce/admin-token-1.png)

![Admin Token 2](/images/bug-forge/daily/gift-lab/administrator-bruteforce/admin-token-2.png)

```
n0MqjBXna9A4zfg
n0MqjBXna9A4bqr
```

The first 12 characters are identical. A wordlist covering all possible 3-character alphanumeric suffixes is generated to cover the full token space.

### Step 3 - Brute Forcing the Admin Token

With the wordlist ready, the `/administrator` request is sent to Caido's Automate tool with the `adminAccessToken` cookie value set as the injection point:

![Automation Configuration](/images/bug-forge/daily/gift-lab/administrator-bruteforce/brute-force.png)

The results are filtered using Caido's response filter to isolate the valid token:

![Denied Results](/images/bug-forge/daily/gift-lab/administrator-bruteforce/brute-force-denied-response.png)

```
resp.raw.ncont:"Denied"
```

![Valid Auth Token](/images/bug-forge/daily/gift-lab/administrator-bruteforce/valid-auth-token.png)

The `Cookie-Editor` browser extension is used to update the `adminAccessToken` cookie to the discovered value:

![Update admin access token](/images/bug-forge/daily/gift-lab/administrator-bruteforce/update-admin-access-token.png)

Refreshing the page grants access to the administrator panel and returns the first flag:

![Flag](/images/bug-forge/daily/gift-lab/administrator-bruteforce/flag.png)

---

### Impact

- An attacker with any valid account can escalate to administrator access by brute forcing a short, predictable token suffix - no knowledge of admin credentials is required
- Once admin access is obtained, full control over application administration functions is possible

---

### Vulnerability Classification

**Broken Authentication - Brute Force**
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Brute Force / Predictable Token
- **Attack Surface:** `adminAccessToken` cookie used to authenticate to `/administrator`
- **CWE:** CWE-307 - Improper Restriction of Excessive Authentication Attempts
- **CWE:** CWE-340 - Generation of Predictable Numbers or Identifiers

---

### Root Cause

The broken authentication vulnerability exists because the `adminAccessToken` is generated with insufficient entropy. The token body is constant across all accounts and only the final 3 characters vary, reducing the effective keyspace to a small brute-forceable range. The `/administrator` endpoint applies no rate limiting or account lockout, allowing an attacker to exhaust all possible values in a short time.

---

### Remediation

- Generate session tokens and access tokens using a cryptographically secure random number generator with sufficient entropy; token values must not be predictable or partially static
- Implement rate limiting and account lockout on all authenticated endpoints, including admin routes, to prevent brute force attacks
- Apply the principle of least privilege so that standard users cannot access administrative functionality even with a valid admin token obtained improperly

---
