---
layout: post
title:  "BugForge - Daily - Shady Oaks Financial"
date:   2026-02-20 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [race-condition,toctou]
categories: [BugForge,daily,shady-oaks-finance]
---

# Daily - Shady Oaks Financial (Repeat)

This is a repeat of the daily challenge from [January 23th, 2026](https://zwarts.dev/posts/2026/01/23/BugForge-Daily-Shady-Oaks-Financial/).

---

## Vulnerability Overview

This challenge demonstrates a race condition vulnerability in a currency exchange endpoint where balance verification and balance deduction are not performed as atomic operations. When multiple concurrent requests target the `/api/convert-currency` endpoint simultaneously, they all read the same account balance before any deductions are applied, allowing each request to proceed as if sufficient funds are available. This Time-of-Check Time-of-Use (TOCTOU) flaw occurs because the application lacks proper synchronization mechanisms such as database-level locking or transactional isolation, enabling an attacker to bypass balance validation checks and perform currency conversions far exceeding their actual available funds.

**Key Issues:**
- Balance verification and balance deduction are not performed as atomic operations
- The application lacks database-level locking or transactional isolation on the `/api/convert-currency` endpoint
- Concurrent requests can all read the same account balance before any deductions are applied, bypassing fund constraints

---

## Vulnerabilities Covered

- Race Condition
- Concurrency Control Bypass (TOCTOU)

---

## Classification

- **OWASP Top 10:** A04:2021 - Insecure Design (lack of concurrency controls and transaction integrity)
- **Vulnerability Type:** Race Condition / Time-of-Check Time-of-Use (TOCTOU)
- **Attack Surface:** Currency conversion API endpoint
- **CWE:** CWE-362 - Concurrent Execution using Shared Resource with Improper Synchronization ('Race Condition')

---
