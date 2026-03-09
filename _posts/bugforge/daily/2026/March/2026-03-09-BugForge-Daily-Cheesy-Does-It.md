---
layout: post
title:  "BugForge - Daily - Cheesy Does It (Repeat)"
date:   2026-03-09 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,type-confusion]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It (Repeat)

This is a repeat of the daily challenge from [January 19th, 2026](https://zwarts.dev/posts/2026/01/19/BugForge-Daily-Cheesy-Does-It/).

---

## Vulnerability Overview

A **Business Logic Flaw** exists in the checkout process where the backend accepts client-supplied `discount` data without enforcing strict validation or business rules. Although individual discount codes are validated, the application fails to handle **array-based** input correctly, allowing an attacker to submit multiple discount codes in a single request and receive cumulative discounts. This results in unauthorized price reductions, potential financial loss, and abuse of promotional mechanisms - highlighting weak server-side enforcement of pricing logic and over-trust in client-controlled request structures.

**Key Issues:**
- The backend accepts array-based input for the discount code parameter without normalization
- Individual discount codes are validated, but multiple codes can be stacked via array injection
- Business rules limiting discount usage are not enforced server-side

---

## Vulnerabilities Covered

- Business Logic Flaw
- Type Confusion (Array Injection)

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Business Logic Flaw / Array Injection
- **Attack Surface:** Checkout and order processing API
- **CWE:** CWE-840 - Business Logic Errors

---
