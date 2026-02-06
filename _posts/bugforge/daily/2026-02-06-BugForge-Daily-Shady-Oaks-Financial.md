---
layout: post
title:  "BugForge - Daily - Shady Oaks Financial (Repeat)"
date:   2026-02-06 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [jwt,authentication-bypass,none-algorithm,broken-authentication]
categories: [BugForge,daily,shady-oaks-financial]
---

# Daily - Shady Oaks Financial (Repeat)

This is a repeat of the daily challenge from
[January 9th, 2026](https://zwarts.dev/posts/2026/01/09/BugForge-Daily-Shady-Oaks-Finance/).

---

## Vulnerability Overview

This challenge demonstrates a **JWT (JSON Web Token) authentication bypass** vulnerability caused by improper algorithm validation. The application accepts JWTs with the `alg` header set to `none`, which instructs the server to skip signature verification entirely. By modifying the JWT header to use the none algorithm and removing the signature portion, an attacker can tamper with the payload claims specifically changing the `role` from `user` to `admin`, without needing to know the secret key used for signing. This allows complete authentication bypass and privilege escalation to administrative access, enabling retrieval of sensitive data from protected admin endpoints.

**Key Issues:**
- The backend JWT library accepts the `none` algorithm, skipping signature verification
- No explicit algorithm validation is enforced during token verification
- The server trusts the `alg` claim from the token header rather than using a server-side whitelist
- Attackers can forge tokens with arbitrary claims including privilege escalation to admin

This represents a classic JWT None Algorithm Attack where insufficient algorithm validation enables complete authentication bypass.

---

## Vulnerabilities Covered

- **JWT None Algorithm Attack** - Exploitation of improper JWT algorithm validation to bypass authentication and escalate privileges
- **Broken Authentication** - Forging unsigned tokens to impersonate administrative users

---

## Classification

- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity

---

