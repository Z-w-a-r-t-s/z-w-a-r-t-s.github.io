---
layout: post
title:  "BugForge - Daily - Copy Pasta (Repeat)"
date:   2026-02-04 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta (Repeat)

This is a repeat of the daily challenge from
[January 7th, 2026](https://zwarts.dev/posts/2026/01/07/BugForge-Daily-CopyPasta/).

---

## Vulnerability Overview

This challenge demonstrates an **Insecure Direct Object Reference (IDOR)** vulnerability in the CopyPasta application's snippet retrieval functionality. The application allows users to create and share code snippets with options to make them public or private.

**Key Issues:**
- The snippet retrieval endpoint (`/api/snippets/:id`) uses sequential integer identifiers
- No authorization checks are performed to verify the requesting user has permission to view the resource
- Any authenticated user can enumerate IDs and access private snippets belonging to other users
- The application relies solely on user-supplied identifiers without validating ownership or visibility

This represents a classic IDOR vulnerability where insufficient access control enables unauthorized access to private resources.

---

## Vulnerabilities Covered

- **Insecure Direct Object Reference (IDOR)** - Exploitation of sequential IDs to access private snippets

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---

