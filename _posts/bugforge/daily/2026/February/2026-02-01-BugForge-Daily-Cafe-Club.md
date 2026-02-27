---
layout: post
title:  "BugForge - Daily - Cafe Club (Repeat)"
date:   2026-02-01 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [brute-force,business-logic-flaw]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club (Repeat)

This is a repeat of the daily challenge from
[December 28th, 2025](https://zwarts.dev/posts/2025/12/28/BugForge-Daily-Cafe-Club/).

---

## Vulnerability Overview

This challenge demonstrates a **business logic flaw involving predictable identifiers and brute force**. Gift card codes are generated with insufficient entropy, where only the final four digits vary between issuances.

**Key Issues:**
- Gift card codes use a partially static structure with only four variable characters
- The small keyspace makes brute force enumeration feasible
- No ownership checks are enforced during gift card redemption
- The redemption endpoint lacks rate limiting and anti-automation controls
- An attacker can systematically enumerate valid codes and redeem gift cards belonging to other users

This represents a fundamental design flaw in token generation and validation that enables unauthorized access to stored value.

---

## Vulnerabilities Covered

- **Brute Force** - Systematic enumeration of the predictable gift card code keyspace
- **Business Logic Flaw** - Insufficient entropy in gift card generation and lack of ownership validation

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-640 - Weak Password Recovery Mechanism for Forgotten Password (applied to weak token generation)

---

