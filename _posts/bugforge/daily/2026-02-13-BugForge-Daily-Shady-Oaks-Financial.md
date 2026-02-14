---
layout: post
title:  "BugForge - Daily - Shady Oaks Financial (Repeat)"
date:   2026-02-13 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control]
categories: [BugForge,daily,shady-oaks-financial]
---

# Daily - Shady Oaks Financial (Repeat)

This is a repeat of the daily challenge from [January 16th, 2026](https://zwarts.dev/posts/2026/01/16/BugForge-Daily-Shady-Oaks-Finance/).


---

## Vulnerability Overview

This challenge demonstrates a **broken access control vulnerability caused by missing function-level authorization**. Administrative endpoints are exposed without proper server-side authorization checks, allowing any authenticated user to directly access privileged functionality and retrieve sensitive data.

**Key Issues:**
- Administrative endpoints (`admin/*`) are accessible without proper authorization checks
- No server-side role validation is performed when accessing privileged routes
- A standard user can directly access `admin/users` and `admin/flag` endpoints
- The backend relies on assumed user roles rather than validating permissions for each request

This represents a fundamental failure of role-based access control enforcement where endpoint access is not restricted based on user privileges.

---

## Vulnerabilities Covered

- **Broken Access Control** - Missing function-level authorization on administrative endpoints

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-285 - Improper Authorization

---
