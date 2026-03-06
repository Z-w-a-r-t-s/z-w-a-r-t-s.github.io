---
layout: post
title:  "BugForge - Daily - Tanuki (Repeat)"
date:   2026-03-03 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [ssrf]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki (Repeat)

This is a repeat of the daily challenge from [February 3rd, 2026](https://zwarts.dev/posts/2026/02/03/BugForge-Daily-Tanuki/).

---

## Vulnerability Overview

A **Server-Side Request Forgery (SSRF)** vulnerability exists in the Tanuki application's leaderboard functionality. The `/api/fetch` endpoint accepts a user-controlled URL parameter to retrieve data from internal services. By intercepting and modifying this request to target `http://localhost:3000/admin` instead of the intended `http://localhost:3000/leaderboard`, an attacker can bypass external access controls and retrieve sensitive administrative data from the internal network.

**Key Issues:**
- The `/api/fetch` endpoint accepts user-controlled URLs without validation or allowlisting
- The server blindly passes client-supplied input to its internal HTTP client
- No restrictions prevent requests to localhost or internal network addresses

---

## Vulnerabilities Covered

- Server-Side Request Forgery (SSRF)

---

## Classification

- **OWASP Top 10:** A10:2021 - Server-Side Request Forgery (SSRF)
- **Vulnerability Type:** Server-Side Request Forgery (SSRF)
- **Attack Surface:** Fetch API endpoint (`/api/fetch`)
- **CWE:** CWE-918 - Server-Side Request Forgery (SSRF)

---
