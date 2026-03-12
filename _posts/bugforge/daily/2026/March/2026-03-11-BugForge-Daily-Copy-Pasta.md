---
layout: post
title:  "BugForge - Daily - Copy Pasta (Repeat)"
date:   2026-03-11 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor,broken-access-control]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta (Repeat)

This is a repeat of the daily challenge from [January 7th, 2026](https://zwarts.dev/posts/2026/01/07/BugForge-Daily-Copy-Pasta/).

---

## Vulnerability Overview

An **IDOR (Insecure Direct Object Reference)** vulnerability exists in the **`snippet retrieval endpoint`** where the application uses sequential integer identifiers without performing authorization checks. Any authenticated user can enumerate snippet IDs and access private snippets belonging to other users. By iterating through IDs using Caido Replay, private snippets containing sensitive data can be retrieved, demonstrating a classic IDOR vulnerability on the read operation.

**Key Issues:**
- The `/api/snippets/:id` endpoint accepts sequential integer IDs without verifying ownership or visibility
- No server-side authorization check confirms the requesting user has permission to view the resource
- The discrepancy between public snippet count and internally assigned IDs reveals the existence of hidden private snippets

---

## Vulnerabilities Covered

- IDOR (Insecure Direct Object Reference) / Broken Access Control

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Snippet retrieval endpoint (`/api/snippets/:id`)
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
