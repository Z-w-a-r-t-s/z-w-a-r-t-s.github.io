---
layout: post
title:  "BugForge - Daily - Cafe Club (Repeat)"
date:   2026-02-22 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli,sqlite]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club (Repeat)

This is a repeat of the daily challenge from [January 11th, 2026](https://zwarts.dev/posts/2026/01/11/BugForge-Daily-Cafe-Club/).

---

## Vulnerability Overview

A **SQL Injection** vulnerability exists in the product API endpoint where the product ID parameter is directly concatenated into a SQLite query without sanitization. By manipulating the `/api/products/{id}` endpoint with a UNION-based SQL injection payload, an attacker can enumerate database schema using SQLite-specific syntax (`sqlite_master` and `pragma_table_info`), extract table and column names, and ultimately retrieve sensitive data including administrator credentials from the users table. The vulnerability allows complete database enumeration and credential theft through crafted SQL payloads.

**Key Issues:**
- The product ID parameter is directly concatenated into the SQL query without sanitization or parameterization
- No input validation enforces that the ID is numeric, allowing arbitrary SQL to be appended
- SQLite-specific schema tables (`sqlite_master`, `pragma_table_info`) are accessible, enabling full database enumeration

---

## Vulnerabilities Covered

- SQL Injection (UNION-based, SQLite)

---

## Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (UNION-based)
- **Attack Surface:** Product API endpoint (`/api/products/{id}`)
- **CWE:** CWE-89 - Improper Neutralization of Special Elements used in an SQL Command

---
