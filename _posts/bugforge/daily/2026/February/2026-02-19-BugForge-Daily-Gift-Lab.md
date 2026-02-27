---
layout: post
title:  "BugForge - Daily - Gift Lab"
date:   2026-02-19 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,idor]
categories: [BugForge,daily,gift-lab]
---

# Daily - Gift Lab
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control - Insecure Direct Object Reference (IDOR)
<br/>
<br/>
<b>Summary:</b>
<br/>
The Gift Lab application contains an **`Insecure Direct Object Reference (IDOR)`** vulnerability in its list sharing functionality. The application generates share links by base64 encoding an identifier string (e.g., `listWithId-2`), a reversible encoding that provides no security guarantee. By decoding the share link, modifying the list identifier to reference another user's list (e.g., `listWithId-1`), and using the manipulated value, an attacker can gain unauthorized read access to any user's wish list. The server performs no ownership or authorization check when resolving the shared list identifier, meaning the base64 encoding acts as a false sense of obscurity rather than a true access control mechanism.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
After registering a new account, explore the application to understand its features. Gift Lab allows users to create **Wish Lists**, add and remove items from those lists, share lists with others, and delete lists. There is no profile management in this version of the application.

![Application UI](/images/bug-forge/daily/gift-lab/ui.png)

---

### Step 2 - Examine the Share Functionality
Navigate to an existing wish list and trigger the **Share** function. Intercept the outgoing request in a proxy tool such as Caido to observe how the share link is constructed.

![Share UI](/images/bug-forge/daily/gift-lab/share-ui.png)

![Share Request](/images/bug-forge/daily/gift-lab/share-request.png)

The share link is a `base64` encoded string. Decode the value to reveal the underlying identifier.

---

### Step 3 - Decode and Manipulate the Share Token
Decode the base64 share token to expose the raw list identifier e.g. `listWithId-2`. Base64 is an encoding scheme, not encryption, so the value can be trivially decoded and modified.

![Decoded Value](/images/bug-forge/daily/gift-lab/base64-decoded.png)

Change the identifier to reference a different list, such as `listWithId-1`, targeting another user's wish list.

![Manipulated Parameter](/images/bug-forge/daily/gift-lab/share-parameter-not-encoded.png)

---

### Step 4 - Access Another User's List
Submit the request with the manipulated identifier. The server resolves the list by ID without verifying whether the requesting user owns it, returning the target list and the flag.

![Flag](/images/bug-forge/daily/gift-lab/flag.png)

---

### Impact
- Any authenticated user can enumerate and access other users' wish lists by incrementing or guessing list identifiers
- Private wish list contents, including personal items and gift preferences, are exposed to unauthorized parties
- The base64 encoding provides a false sense of security, as it is trivially reversible and offers no cryptographic protection
- An attacker could systematically scrape all lists on the platform by iterating through sequential IDs
- Undermines user trust and violates the expected privacy of personal wish lists

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** List share endpoint, base64-encoded list identifier parameter
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key
- **CWE:** CWE-284 - Improper Access Control

---

### Root Cause
The share functionality constructs a token by base64 encoding a predictable, sequential list identifier (e.g., `listWithId-2`). When the share link is resolved, the server decodes the token and fetches the corresponding list directly by ID without verifying that the requesting user is the owner or an authorized recipient. Base64 encoding is not a security control, it is a data encoding format that any user can decode and manipulate. The application relies on obscurity rather than enforcing server-side ownership validation, making every list on the platform accessible to any user who understands the token format.

---

### Remediation
- Implement server-side authorization checks on all list retrieval endpoints to verify the requesting user owns or has been explicitly granted access to the requested list
- Replace sequential, guessable list identifiers with cryptographically random, unguessable tokens (e.g., UUIDs or signed tokens) so that IDs cannot be enumerated
- Never rely on encoding schemes such as base64 to protect sensitive references, encoding is not encryption
- If sharing is intentional, generate short-lived, single-use share tokens that are stored server-side and tied to a specific list, rather than encoding the list ID directly
- Audit all endpoints that accept user-supplied identifiers to ensure ownership and access checks are consistently enforced

---
