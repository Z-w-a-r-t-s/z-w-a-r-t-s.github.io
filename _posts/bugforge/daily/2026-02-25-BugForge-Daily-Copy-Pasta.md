---
layout: post
title:  "BugForge - Daily - Copy Pasta (Repeat)"
date:   2026-02-25 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta (Repeat)

This is a repeat of the daily challenge from [January 21th, 2026](https://zwarts.dev/posts/2026/01/21/BugForge-Daily-Copy-Pasta/).

---

## Vulnerability Overview

A **Broken Access Control** vulnerability exists in the Copy Pasta application's `password reset endpoint`. The endpoint accepts a `userId` parameter without verifying that the authenticated user is authorized to act on the referenced account, allowing an attacker to reset passwords for arbitrary users, including administrators. Public API responses further leak usernames, making it trivial to identify high-value targets. By chaining identifier manipulation with information disclosure, an attacker can achieve full account takeover and access sensitive application data.

**Key Issues:**
- The password reset endpoint trusts client-supplied `userId` without enforcing ownership checks on the authenticated session
- Usernames are exposed in responses from the `/api/snippets/public` endpoint, enabling attacker enumeration of high-value accounts
- No object-level authorization prevents horizontal or vertical privilege escalation on account management functionality

---

## Vulnerabilities Covered

- Broken Access Control
- Insecure Direct Object Reference (IDOR)

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** User profile and password reset endpoints
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
