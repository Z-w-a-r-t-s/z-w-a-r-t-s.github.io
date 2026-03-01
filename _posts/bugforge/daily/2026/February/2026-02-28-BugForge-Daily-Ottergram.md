---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-02-28 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,idor,verb-tampering]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram
><br/><b>Vulnerabilities Covered:</b>
<br/>
HTTP Verb Tampering<br/>
Broken Access Control - Insecure Direct Object Reference (IDOR)
<br/>
<br/>
<b>Summary:</b>
<br/>
The Ottergram application contains an **`HTTP Verb Tampering`** vulnerability combined with an **`Insecure Direct Object Reference (IDOR)`** flaw in its `comment` functionality. After exploring the application's comment feature, the standard `POST` request used to add a comment was identified as the attack surface. By changing the HTTP method from `POST` to `PUT` and appending a target comment ID to the request URL, the server unexpectedly processes the request and updates the specified comment without verifying ownership. The application exposes an undocumented verb-based route that applies no authorisation check, meaning any user can overwrite any comment simply by supplying its ID. This demonstrates a critical failure where HTTP method handling is inconsistent and resource-level access control is entirely absent.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Reconnaissance

Begin by mapping the application's attack surface. Browse the Ottergram feed and interact with posts, paying particular attention to the comment functionality. Proxy all traffic to capture the underlying API requests the application makes when submitting a comment.

![Original POST to add comment](/images/bug-forge/daily/ottergram/update-comment-verb-tamp/add-comment-post.png)

Intercepting the comment submission reveals a `POST` request to the comments endpoint, with the comment body supplied in the request payload. The endpoint accepts a user-supplied comment and associates it with a post. Note that the URL structure does not include a comment ID at this stage, this becomes significant in the next step.

---

### Step 2 - Apply HTTP Verb Tampering

With the `POST` request captured in the proxy, modify two things: change the HTTP method from `POST` to `PUT`, and append a target comment ID directly to the request URL (e.g. `/api/comments/1`). Supply an updated comment body in the payload.

![PUT Request - Tampered](/images/bug-forge/daily/ottergram/update-comment-verb-tamp/commen-put-request.png)

The server accepts the `PUT` request and processes it without error. Rather than rejecting the unsupported verb or enforcing an ownership check against the authenticated user, the application routes the request to an undocumented update handler. The comment ID in the URL is trusted directly, meaning any comment on the platform can be targeted by supplying its ID, with no ownership of the resource required.

---

### Step 3 - Verify the Modification

Retrieve all comments for Post ID 1 to confirm the update was applied.

![Get - Post comments](/images/bug-forge/daily/ottergram/update-comment-verb-tamp/get-comments-from-posts.png)

The response confirms that the comment has been overwritten with the attacker-supplied content. The flag is visible inside the updated comment, demonstrating that a resource belonging to another user has been successfully modified without any authorisation check.

---

### Impact
- Any authenticated user can overwrite any comment on the platform by changing the HTTP method to `PUT` and supplying a sequential comment ID
- Comment content authored by other users can be modified or defaced without their knowledge or consent
- Undocumented HTTP verb routes are exposed server-side with no equivalent authorisation enforcement to the primary `POST` handler
- Sequential integer IDs make enumeration trivial, allowing an attacker to iterate through comment IDs to modify comments at scale
- Injecting malicious content into other users' comments could be used as a vector for social engineering or reputational damage

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** HTTP Verb Tampering / Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Comment endpoint (`PUT /api/comments/{id}`)
- **CWE:** CWE-650 - Trusting HTTP Permission Methods on the Server Side
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The server exposes an undocumented `PUT` route on the comment endpoint that was likely introduced during development but never properly secured. When the `PUT` method is used with a comment ID in the URL path, the server routes the request to an update handler that performs no ownership verification; it does not check whether the authenticated user is the author of the comment being modified. The application blindly trusts the user-supplied comment ID as the target resource, creating a classic IDOR condition. The absence of consistent access control enforcement across all HTTP verb handlers is the fundamental root cause.

---

### Remediation
- Enforce ownership checks on all comment modification endpoints: verify that the authenticated user's ID matches the `author_id` of the comment record before processing any update, returning `403 Forbidden` if the check fails
- Explicitly define and allowlist permitted HTTP methods on each route; return `405 Method Not Allowed` for any verb not intentionally supported
- Audit all API routes to ensure every HTTP verb handler applies the same authorisation controls; a secured `POST` handler is not sufficient if a `PUT` or `PATCH` handler on the same resource is left unprotected
- Replace sequential integer comment IDs with opaque, non-enumerable identifiers (e.g. UUIDs) to prevent mass exploitation via ID enumeration
- Apply the principle of least privilege: a user should only be permitted to modify resources they own, regardless of the HTTP method used
- Include verb-tampering and IDOR test cases in the application's security test suite to prevent regressions

---
