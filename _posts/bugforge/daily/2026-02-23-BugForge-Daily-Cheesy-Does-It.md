---
layout: post
title:  "BugForge - Daily - Cheesy Does It"
date:   2026-02-23 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It (Repeat)

This is a repeat of the daily challenge from [January 12th, 2026](https://zwarts.dev/posts/2026/01/12/BugForge-Daily-Cheesy-Does-It/).

---

## Vulnerability Overview

A **Business Logic Flaw** exists in the checkout process where the `POST /order` endpoint trusts client-supplied pricing data. By intercepting the checkout request and modifying the unit price and total price fields to zero, an attacker can place a valid order that is accepted and processed by the server at `$0`. The backend performs no server-side price validation or recalculation against authoritative product data, making it trivially exploitable.

**Key Issues:**
- The `POST /order` request includes client-controlled pricing fields (unit price and total price)
- The backend accepts and processes these values without recalculating or validating them server-side
- No integrity checks exist to detect abnormal or zero-value orders before processing

---

## Vulnerabilities Covered

- Business Logic Flaw (Client-Side Price Manipulation)

---

## Classification

- **OWASP Top 10:** A04:2021 - Insecure Design
- **Vulnerability Type:** Business Logic Flaw (Client-Side Price Manipulation)
- **Attack Surface:** Checkout endpoint (`POST /order`)
- **CWE:** CWE-602 - Client-Side Enforcement of Server-Side Security

---
