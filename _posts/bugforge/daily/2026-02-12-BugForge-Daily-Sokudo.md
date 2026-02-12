---
layout: post
title:  "BugForge - Daily - Sokudo (Repeat)"
date:   2026-02-12 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-authentication]
categories: [BugForge,daily,sokudo]
---

# Daily - Sokudo (Repeat)

This is a repeat of the daily challenge from
[Jan 29th, 2026](https://zwarts.dev/posts/2026/01/29/BugForge-Daily-Sokudo/).

---

## Vulnerability Overview

The Sokudo application uses predictable ISO 8601 timestamps as authentication tokens, creating a critical broken authentication vulnerability. The Bearer token is simply the user's registration timestamp in `YYYYMMDDHHmmss` format. Combined with an information disclosure vulnerability in the leaderboard endpoint that exposes each user's `last_login` timestamp, an attacker can derive any user's authentication token. By converting the admin user's last login time to the ISO 8601 format and using it as a Bearer token, complete administrative access was achieved, demonstrating how predictable tokens and excessive data exposure can lead to full account takeover.

**Key Issues:**
- The application generates authentication tokens using predictable timestamps rather than cryptographically secure random values
- The Bearer token is the user's registration or login timestamp converted to ISO 8601 format (`YYYYMMDDHHmmss`)
- The leaderboard endpoint (`/api/stats`) returns excessive user information including the `last_login` timestamp
- Attackers can compute valid authentication tokens for any user, including administrators

This represents a broken authentication vulnerability where predictable token generation combined with information disclosure enables complete account takeover.

---

## Vulnerabilities Covered

- **Broken Authentication / Predictable Tokens** - Authentication tokens derived from predictable ISO 8601 timestamps enabling token forgery and account takeover
- **Information Disclosure** - Leaderboard endpoint exposes user `last_login` timestamps, providing attackers the data needed to compute valid tokens

---

## Classification

- **OWASP Top 10:** A07:2021 - Identification and Authentication Failures
- **CWE:** CWE-330 - Use of Insufficiently Random Values, CWE-200 - Exposure of Sensitive Information

---
