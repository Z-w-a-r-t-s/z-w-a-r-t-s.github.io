---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2026-02-08 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
Insecure Direct Object Reference (IDOR)<br/>
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge exploits an **Insecure Direct Object Reference (IDOR)** vulnerability in the profile password update functionality. The application includes the user's ID in the password change request payload without verifying that the authenticated user is authorized to modify that account. By intercepting the request and changing the user ID, an attacker can overwrite another user's password, gaining unauthorized access to arbitrary accounts including the admin.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis and Feature Enumeration

Register an account and log in to explore the CafeClub application with Caido configured to intercept traffic. The application functions as an e-commerce platform with product browsing, shopping cart, checkout, gift cards, and a loyalty points system.

Test the ordering workflow including cart manipulation, gift card redemption, and points usage. No new vulnerabilities were identified in the checkout flow, so attention shifted to the profile management features.

---

### Step 2 - Profile Functionality Analysis

Navigate to the user profile section and explore the available features. The profile includes a password update function. Intercept the password change request using Caido and analyze the request payload.

The password update request includes the user's ID as a parameter in the body alongside the new password. This is a strong indicator of a potential IDOR vulnerability, as the server appears to rely on the client-supplied ID to determine which account to update rather than deriving it from the authenticated session.

---

### Step 3 - IDOR Exploitation

Send the intercepted password change request to Caido's Replay. Modify the user ID parameter in the payload from the current user's ID to `1`, which is conventionally the first account created and typically belongs to the administrator.

Submit the modified request. The server processes the password change without verifying that the authenticated user owns the account referenced by the supplied ID. The admin account's password is successfully overwritten with the attacker-controlled value, revealing the flag.

![Flag](/images/bug-forge/daily/cafe-club/idor/flag.png)

---

### Impact
- Unauthorized modification of any user's password by manipulating the user ID parameter
- Full account takeover of arbitrary accounts including administrator accounts
- Complete compromise of user account security across the platform
- Potential for privilege escalation by targeting administrative accounts
- Ability to lock legitimate users out of their accounts
- Exposure of sensitive user data accessible after account takeover
- Undermines the integrity of the entire authentication system

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Profile password update API endpoint
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

### Root Cause
The backend accepts a user-supplied ID in the password change request payload and uses it to determine which account to update, without verifying that the authenticated user is authorized to modify that account. The server fails to enforce object-level authorization, trusting the client-provided ID instead of deriving the target account from the authenticated session token.

This is a classic IDOR vulnerability where the developer assumed users would only submit their own ID through the intended UI, ignoring the possibility of request manipulation through intercepting proxies. The password update endpoint lacks any server-side ownership validation between the session identity and the target user ID.

---

### Remediation
- Derive the target user ID from the authenticated session on the server side rather than accepting it from the client
- Remove the user ID parameter from the password change request payload entirely
- Implement object-level authorization checks to verify that the authenticated user owns the resource being modified
- Apply the principle of least privilege by ensuring users can only modify their own account data
- Add audit logging for all password change operations to detect unauthorized modifications
- Implement re-authentication (current password verification) before allowing password changes
- Conduct regular access control testing to identify IDOR vulnerabilities across all API endpoints
- Apply a consistent authorization framework across all endpoints that reference user-controlled identifiers

---
