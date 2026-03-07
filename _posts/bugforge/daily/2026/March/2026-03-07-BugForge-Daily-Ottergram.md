---
layout: post
title:  "BugForge - Daily - Ottergram (Repeat)"
date:   2026-03-07 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram (Repeat)

This is a repeat of the daily challenge from [January 3rd, 2026](https://zwarts.dev/posts/2026/01/03/BugForge-Daily-Ottergram/).

---

## Vulnerability Overview

A **SQL Injection** vulnerability exists in the `user profile` retrieval endpoint where user-supplied input is concatenated directly into SQL queries without parameterization or input validation. By injecting a single quote to confirm the vulnerability, using `ORDER BY` to enumerate columns, and constructing a UNION-based payload, an attacker can extract sensitive data from the underlying database - including administrator credentials and the challenge flag.

**Key Issues:**
- User input is concatenated into SQL queries without sanitization
- The number of columns can be enumerated via `ORDER BY` to craft a valid UNION payload
- Sensitive tables and credentials are accessible through `sqlite_master` and targeted UNION queries

---

## Vulnerabilities Covered

- SQL Injection (UNION-based)

---

## Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (UNION-based)
- **Attack Surface:** Profile retrieval endpoint with unsanitized user input
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---
