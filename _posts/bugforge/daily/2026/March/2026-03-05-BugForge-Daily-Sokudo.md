---
layout: post
title:  "BugForge - Daily - Sokudo (Repeat)"
date:   2026-03-05 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,http-verb-tampering]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo (Repeat)

This is a repeat of the daily challenge from [January 22nd, 2026](https://zwarts.dev/posts/2026/01/22/BugForge-Daily-Sokudo/).

---

## Vulnerability Overview

A **Broken Access Control** vulnerability exists in the `/api/stats` endpoint where the server enforces authorization on GET requests but fails to apply the same checks to the PUT method. By leveraging HTTP verb tampering and switching from GET to PUT while manipulating the `user_id` parameter, an attacker can update typing statistics without authorization.

**Key Issues:**
- Authorization checks are only applied to the GET method, leaving PUT unprotected
- The `user_id` parameter can be manipulated to reference arbitrary users
- No consistent authorization enforcement across all supported HTTP methods

---

## Vulnerabilities Covered

- Broken Access Control
- HTTP Verb Tampering

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** HTTP Verb Tampering (HTTP Method Override)
- **Attack Surface:** API endpoint accepting multiple HTTP methods without consistent authorization
- **CWE:** CWE-650 - Trusting HTTP Permission Methods on the Server Side

---
