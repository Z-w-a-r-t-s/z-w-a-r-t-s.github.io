---
layout: post
title:  "BugForge - Daily - Tanuki (Repeat)"
date:   2026-02-25 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki (Repeat)

This is a repeat of the daily challenge from [January 13th, 2026](https://zwarts.dev/posts/2026/01/13/BugForge-Daily-Tanuki/).

---

## Vulnerability Overview

An **Insecure Direct Object Reference (IDOR)** vulnerability exists in the Tanuki flashcard application's statistics API endpoint. After registering an account and being assigned a sequential user ID, the `/api/stats/{id}` endpoint lacks proper authorization controls. By manipulating the numeric ID parameter, an attacker can access other users' statistics and sensitive data — including a hidden achievement flag — without any ownership verification.

**Key Issues:**
- The `/api/stats/{id}` endpoint authenticates users but does not verify that the authenticated user owns the requested resource
- Sequential numeric user IDs are exposed during registration, making enumeration trivial
- No object-level access control exists to prevent horizontal privilege escalation

---

## Vulnerabilities Covered

- Insecure Direct Object Reference (IDOR)

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** User statistics API endpoint (`GET /api/stats/{id}`)
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
