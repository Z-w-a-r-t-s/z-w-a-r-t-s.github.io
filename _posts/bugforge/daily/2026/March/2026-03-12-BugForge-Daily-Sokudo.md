---
layout: post
title:  "BugForge - Daily - Sokudo (Repeat)"
date:   2026-03-12 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo (Repeat)

This is a repeat of the daily challenge from [January 29th, 2026](https://zwarts.dev/posts/2026/01/29/BugForge-Daily-Sokudo/).

---

## Vulnerability Overview

The Sokudo application uses predictable ISO 8601 timestamps as authentication tokens, creating a critical broken authentication vulnerability. By analyzing the application's responses, it was discovered that the Bearer token is simply the user's registration timestamp in `YYYYMMDDHHmmss` format. Combined with an information disclosure vulnerability in the leaderboard endpoint that exposes each user's `last_login` timestamp, an attacker can derive any user's authentication token. By converting the admin user's last login time to the ISO 8601 format and using it as a Bearer token, complete administrative access was achieved, demonstrating how predictable tokens and excessive data exposure can lead to full account takeover.

**Key Issues:**
- Authentication tokens are generated using predictable timestamps rather than cryptographically secure random values
- The leaderboard endpoint overshares user data including `last_login` timestamps
- The combination of predictable token generation and information disclosure creates a complete authentication bypass

---

## Vulnerabilities Covered

- Broken Authentication / Predictable Tokens

---

## Classification

- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **Vulnerability Type:** Broken Authentication / Predictable Tokens
- **Attack Surface:** Authentication token generation, Leaderboard API endpoint (`/api/stats`)
- **CWE:** CWE-330 - Use of Insufficiently Random Values, CWE-200 - Exposure of Sensitive Information

---
