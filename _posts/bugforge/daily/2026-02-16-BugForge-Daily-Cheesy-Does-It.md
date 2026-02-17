---
layout: post
title:  "BugForge - Daily - Cheesy Does It (Repeat)"
date:   2026-02-16 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It (Repeat)

This is a repeat of the daily challenge from [January 5th, 2026](https://zwarts.dev/posts/2026/01/05/BugForge-Daily-Cheesy-Does-It/).

---

## Vulnerability Overview

The login functionality is vulnerable to SQL Injection, allowing attackers to bypass authentication and gain unauthorized access to admin accounts. By injecting a payload such as `' or 1=1--` into the username field, the attacker manipulates the underlying SQL query to always evaluate as true, effectively bypassing password validation. The backend constructs SQL queries by directly concatenating user-supplied input without proper sanitization or parameterization, allowing attackers to inject malicious SQL syntax that alters the query logic and bypasses authentication controls.

**Key Issues:**
- The login endpoint directly concatenates user input into SQL queries without parameterization
- No input validation or sanitization is applied to authentication fields
- SQL injection in the username field allows complete authentication bypass
- Attackers gain unauthorized access to administrative accounts without valid credentials

This represents a classic SQL injection vulnerability where insufficient input handling enables complete authentication bypass.

---

## Vulnerabilities Covered

- **SQL Injection (Authentication Bypass)** - The login API endpoint constructs SQL queries using unsanitized user input, allowing attackers to inject payloads that bypass password validation and gain unauthorized access to privileged accounts

---

## Classification

- **OWASP Top 10:** A03:2021 - Injection
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---
