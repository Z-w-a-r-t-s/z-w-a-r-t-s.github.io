---
layout: post
title:  "BugForge - Daily - Shady Oaks Finance (Repeat)"
date:   2026-03-06 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [jwt,authentication-bypass,none-algorithm,broken-authentication]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Finance (Repeat)

This is a repeat of the daily challenge from [January 9th, 2026](https://zwarts.dev/posts/2026/01/09/BugForge-Daily-Shady-Oaks-Finance/).

---

## Vulnerability Overview

A **JWT None Algorithm Attack** exists in the authentication system where the server accepts JWTs with the `alg` header set to `none`, skipping signature verification entirely. By modifying the JWT header to use the none algorithm and removing the signature, an attacker can tamper with payload claims - changing the `role` from `user` to `admin` - without knowing the secret key. This allows complete authentication bypass and privilege escalation to administrative access.

**Key Issues:**
- The server accepts the `none` algorithm, allowing unsigned tokens to be processed
- JWT payload claims such as `role` are trusted without signature verification
- No server-side algorithm whitelist is enforced during token validation

---

## Vulnerabilities Covered

- JWT None Algorithm Attack
- Broken Authentication

---

## Classification

- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** JWT None Algorithm Attack / Broken Authentication
- **Attack Surface:** Authentication token handling
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity

---
