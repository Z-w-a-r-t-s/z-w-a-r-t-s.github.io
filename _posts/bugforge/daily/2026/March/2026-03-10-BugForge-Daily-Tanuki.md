---
layout: post
title:  "BugForge - Daily - Tanuki (Repeat)"
date:   2026-03-10 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [mass-assignment]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki (Repeat)

This is a repeat of the daily challenge from [December 30th, 2025](https://zwarts.dev/posts/2025/12/30/BugForge-Daily-Tanuki/).

---

## Vulnerability Overview

A **Mass Assignment** vulnerability exists in the **`user registration flow`** where the application trusts client-supplied input and allows sensitive attributes such as `user_role` to be set directly by the user. By manipulating the registration request and changing the role from `user` to `admin`, an attacker can create an account with elevated privileges. The absence of server-side allowlisting and role enforcement results in a complete breakdown of authorization controls and unrestricted access to administrative functionality.

**Key Issues:**
- The `user_role` parameter is included in the client-side registration payload and passed directly to the backend
- No server-side allowlist or enforcement restricts which fields may be modified during account creation
- Sensitive attributes such as roles are not assigned exclusively on the backend

---

## Vulnerabilities Covered

- Mass Assignment / Privilege Escalation

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Mass Assignment / Privilege Escalation
- **Attack Surface:** User registration API
- **CWE:** CWE-915 - Improperly Controlled Modification of Dynamically-Determined Object Attributes

---
