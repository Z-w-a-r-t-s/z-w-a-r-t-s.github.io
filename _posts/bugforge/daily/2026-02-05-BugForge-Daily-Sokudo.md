---
layout: post
title:  "BugForge - Daily - Sokudo (Repeat)"
date:   2026-02-05 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo (Repeat)

This is a repeat of the daily challenge from
[January 1st, 2026](https://zwarts.dev/posts/2026/01/01/BugForge-Daily-Sokudo/).

---

## Vulnerability Overview

This challenge demonstrates a **Broken Access Control** vulnerability through **HTTP Verb Tampering** in the Sokudo typing application's statistics endpoint. The application allows users to complete typing tests and track their performance statistics.

**Key Issues:**
- The `/api/stats` endpoint implements access restrictions for POST but not for PUT
- No consistent authorization checks are applied across all HTTP methods
- Attackers can bypass access controls by simply changing the HTTP verb
- The server trusts the HTTP method context without enforcing uniform authorization logic

This represents a classic HTTP Verb Tampering vulnerability where inconsistent security enforcement enables unauthorized modification of resources.

---

## Vulnerabilities Covered

- **Broken Access Control** - Exploitation of HTTP verb tampering to bypass authorization and manipulate statistics

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-650 - Trusting HTTP Permission Methods on the Server Side

---

