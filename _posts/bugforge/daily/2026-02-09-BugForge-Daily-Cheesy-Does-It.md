---
layout: post
title:  "BugForge - Daily - Cheesy Does It (Repeat)"
date:   2026-02-09 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It (Repeat)

This is a repeat of the daily challenge from
[December 29th, 2025](https://zwarts.dev/posts/2025/12/29/BugForge-Daily-Cheesy-Does-It/).

---

## Vulnerability Overview

This vulnerability is an **Insecure Direct Object Reference (IDOR)** caused by missing server-side authorization checks when accessing order data. The application exposes predictable, sequential `order_id` values and trusts the client-supplied identifier without verifying that the authenticated user owns the referenced order. By enumerating and manipulating the `order_id` parameter, an attacker can access other users' orders, resulting in horizontal privilege escalation and exposure of sensitive data.

**Key Issues:**
- The application uses sequential, predictable `order_id` values
- No server-side authorization checks verify user ownership of the requested order
- The backend trusts the client-supplied `order_id` parameter without validation
- Attackers can enumerate and access any user's order data through parameter manipulation

This represents a classic IDOR vulnerability where insufficient access control enables horizontal privilege escalation.

---

## Vulnerabilities Covered

- **IDOR (Insecure Direct Object Reference)** - Exploitation of predictable order identifiers to access other users' data without authorization
- **Broken Access Control** - Missing server-side checks allowing horizontal privilege escalation across user accounts

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
