---
layout: post
title:  "BugForge - Weekly - Galaxy Dash"
date:   2026-02-20 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [xss,session-hijacking,account-takeover]
categories: [BugForge,weekly,galaxy-dash]
---

# Weekly - Galaxy Dash
><br/><b>Vulnerabilities Covered:</b>
<br/>
Stored XSS (Second-Order) via `cargo_size` Parameter
<br/>
Session Hijacking via localStorage JWT Token Extraction
<br/>
Account Takeover via Support User Token Theft
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a stored cross-site scripting (XSS) vulnerability in a delivery booking application where unsanitized user input in the `cargo_size` field is persisted server-side and reflected without sanitization when an order invoice is rendered, classifying this as second-order XSS. A crafted payload that reads the viewer's JWT token from localStorage and exfiltrates it to an external endpoint was injected at booking creation. Because the application stores JWT tokens in localStorage and exposes a support request feature, the malicious booking was submitted for support review. When the support agent opened the invoice, their session token was silently captured and used to authenticate as the support user, with the challenge flag returned in the `X-Flag` response header.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis & Directory Enumeration

Inspecting the authentication flow, the application stores the JWT token in localStorage rather than an HttpOnly cookie, making it accessible to any JavaScript executing on the page.

![JWT Token stored in local storage](/images/bug-forge/weekly/galaxy-dash/support-xss/local-storage-token.png)

Testing the order and delivery functionality revealed a new feature: `Request Support`, available from the delivery list.

![Support UI](/images/bug-forge/weekly/galaxy-dash/support-xss/support-ui.png)

---

### Step 2 - Identifying the XSS Injection Point

Testing the delivery booking for injection vulnerabilities, the `cargo_size` field was found to be vulnerable to XSS. The payload is not executed at the point of injection, it is only triggered when the order invoice is opened. This deferred execution is characteristic of **second-order XSS**: the payload is stored in one context (the booking creation request) and executed in a separate context (the invoice render), typically triggered by a different user action or a different user entirely.

![XSS Post Request](/images/bug-forge/weekly/galaxy-dash/support-xss/post-booking-xss-request.png)

![XSS NO trigger](/images/bug-forge/weekly/galaxy-dash/support-xss/xss-no-trigger-ui.png)

![Invoice XSS Trigger](/images/bug-forge/weekly/galaxy-dash/support-xss/invoice-xss-triggered.png)

---

### Step 3 - Extracting the JWT Token via XSS

With confirmed XSS execution on the invoice page, a payload was crafted to read the JWT token from `localStorage` and exfiltrate it to an external listener on [webhook.site](https://webhook.site).

![XSS - Local Storage](/images/bug-forge/weekly/galaxy-dash/support-xss/xss-token-extraction.png)

The payload successfully extracted the token and sent it as a callback to webhook.site, confirming that any user who views the invoice will have their session token captured.

![XSS - Token Extracted](/images/bug-forge/weekly/galaxy-dash/support-xss/xss-token-extracted.png)

![Webhook.site Post Payload - To Extract Token](/images/bug-forge/weekly/galaxy-dash/support-xss/webhook-site-post.png)

Navigating to the invoice for order `11` triggered the payload, and the token was received on webhook.site. Confirming end-to-end exfiltration.

---

### Step 4 - Targeting the Support User

Using the `Request Support` feature, the malicious delivery was submitted for review. When the support agent opened the invoice, the second-order XSS payload executed in their browser context and sent their JWT token to webhook.site.

![Support Modal](/images/bug-forge/weekly/galaxy-dash/support-xss/support-modal.png)

The support user's token was received on webhook.site.

![Support User Token](/images/bug-forge/weekly/galaxy-dash/support-xss/webhook-support-token.png)

---

### Step 5 - Session Hijacking & Flag Retrieval

The captured token was placed into localStorage, replacing the current session. After refreshing the page, the session was authenticated as the support user.

![Logged In as support user](/images/bug-forge/weekly/galaxy-dash/support-xss/support-local-token.png)

No flag was visible in the UI, but inspecting the booking API response revealed the `X-Flag` header containing the challenge flag.

![Flag](/images/bug-forge/weekly/galaxy-dash/support-xss/flag.png)

---

### Impact

- Account takeover of privileged internal users (support agents) via session token theft
- Complete bypass of authentication and access controls through stolen JWT credentials
- Exposure of any data accessible to the compromised support account, including internal delivery records
- Silent, user-interaction-triggered exploitation with no visible indicators to the victim
- Potential for lateral movement to other application features or tenants accessible from the support role

---

### Vulnerability Classification

- **OWASP Top 10:** A03:2021 - Injection / A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Stored XSS (Second-Order) leading to Session Hijacking and Account Takeover
- **Attack Surface:** Delivery booking `cargo_size` parameter; order invoice render; support request workflow
- **CWE:**
  - CWE-79 - Improper Neutralization of Input During Web Page Generation (Cross-site Scriating)
  - CWE-1004 - Sensitive Cookie Without 'HttpOnly' Flag (analogous misuse: JWT stored in accessible localStorage)
  - CWE-384 - Session Fixation

---

### Root Cause

The application fails to sanitize or encode user-supplied input stored in the `cargo_size` field before reflecting it in the invoice HTML. Because the payload executes in a different rendering context than where it was submitted, standard reflected XSS mitigations (e.g., per-request output encoding checks) are insufficient without addressing the stored value. Compounding the issue, the application stores JWT session tokens in localStorage rather than HttpOnly cookies, making them accessible to any JavaScript running on the page. The combination of unsanitized stored output and token storage in a JavaScript-accessible location creates a complete exploitation chain from XSS to account takeover.

---

### Remediation

- Sanitize and HTML-encode all user-supplied input before storing it, and again before rendering it in any HTML context, including invoice templates
- Store JWT tokens in HttpOnly, Secure, SameSite cookies rather than localStorage to prevent JavaScript access
- Implement a strict Content Security Policy (CSP) to restrict script execution sources and block inline scripts
- Apply input validation on the `cargo_size` field to reject unexpected characters or payloads at the API boundary
- Audit all stored data rendering paths to identify other second-order XSS opportunities where stored values are reflected in separate contexts
- Add automated security scanning for XSS in stored fields as part of the CI/CD pipeline

---
