---
layout: post
title:  "BugForge - Daily - Cafe Club (Repeat)"
date:   2026-03-08 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [brute-force,business-logic-flaw]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club (Repeat)

This is a repeat of the daily challenge from [December 28th, 2025](https://zwarts.dev/posts/2025/12/28/BugForge-Daily-Cafe-Club/).

---

## Vulnerability Overview

A **Business Logic Flaw** exists in the `gift card generation and redemption` system where codes are issued with insufficient entropy. By comparing purchased gift cards, only the final four letters varied between issuances while the remainder remained static. Using a generated wordlist covering the entire four-character keyspace, an attacker can brute force the redemption endpoint to enumerate valid codes belonging to other users. The absence of ownership checks, rate limiting, and anti-automation controls allows this attack to succeed at scale.

**Key Issues:**
- Gift card codes are generated with insufficient randomness, leaving only four characters as the variable portion
- The redemption endpoint does not enforce ownership checks, allowing codes issued to other users to be redeemed
- No rate limiting or anti-automation protections are in place on the redemption endpoint

---

## Vulnerabilities Covered

- Brute Force
- Business Logic Flaw (Predictable Identifier)

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Business Logic Flaw / Predictable Identifier
- **Attack Surface:** Gift card redemption API with insufficient code entropy and no ownership validation
- **CWE:** CWE-640 - Weak Password Recovery Mechanism for Forgotten Password (applied to weak token generation)

---
