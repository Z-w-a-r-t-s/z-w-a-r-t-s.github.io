---
layout: post
title:  "BugForge - Daily - Copy Pasta (Repeat)"
date:   2026-03-04 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta (Repeat)

This is a repeat of the daily challenge from [January 28th, 2026](https://zwarts.dev/posts/2026/01/28/BugForge-Daily-Copy-Pasta/).

---

## Vulnerability Overview

A **SQL Injection** vulnerability exists in the CopyPasta application's `/api/snippets/share/:id` endpoint, where the GUID parameter is directly concatenated into SQL queries without sanitization. By injecting SQL payloads into the share endpoint, an attacker can perform UNION-based SQL injection to enumerate the SQLite database structure, extract table names and column schemas, and ultimately dump sensitive data including all usernames and passwords from the users table, resulting in complete database compromise and potential account takeover.

**Key Issues:**
- The GUID parameter in the share endpoint is concatenated directly into SQL queries without sanitization
- No input validation rejects unexpected characters in the GUID parameter
- The SQLite database processes injected input as part of the query structure rather than as data

---

## Vulnerabilities Covered

- SQL Injection (Union-Based)

---

## Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (Union-Based)
- **Attack Surface:** Snippet share endpoint (`/api/snippets/share/:id`)
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---
