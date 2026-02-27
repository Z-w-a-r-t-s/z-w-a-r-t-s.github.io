---
layout: post
title:  "BugForge - Daily - Sokudo (Repeat)"
date:   2026-02-26 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [api-versioning,broken-authentication,idor,jwt-manipulation]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo (Repeat)

This is a repeat of the daily challenge from [January 15th, 2026](https://zwarts.dev/posts/2026/01/15/BugForge-Daily-Sokudo/).

---

## Vulnerability Overview

A **Broken Authentication** and **JWT Signature Bypass** vulnerability exists in the Sokudo application's legacy API. The application exposes both `/v1` and `/v2` API versions, where `/v2` enforces proper JWT validation while the legacy `/v1` endpoints fail to verify token signatures or role claims. By discovering the `/v1/admin/flag` endpoint through enumeration and crafting a forged JWT using the `alg: none` attack (removing the signature entirely and elevating the `role` claim to `"admin"`), an attacker can bypass authentication and access administrative functionality without valid credentials.

**Key Issues:**
- Legacy `/v1` endpoints accept JWT tokens without verifying the cryptographic signature, enabling full token forgery
- No security parity between API versions allows attackers to downgrade to the unprotected `/v1` surface
- Authorization relies solely on token claims (`role`, `user_id`) without server-side verification, enabling IDOR on account context

---

## Vulnerabilities Covered

- API Versioning Vulnerability
- Broken Authentication
- Insecure Direct Object Reference (IDOR)
- JWT Token Manipulation (Algorithm None Attack)

---

## Classification

- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Broken Authentication, JWT Signature Bypass
- **Attack Surface:** Legacy `/v1` admin API endpoints
- **CWE:** CWE-287 - Improper Authentication
- **CWE:** CWE-345 - Insufficient Verification of Data Authenticity
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
