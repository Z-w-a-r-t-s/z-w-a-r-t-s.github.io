---
layout: post
title:  "BugForge - Daily - Cheesy Does It (Repeat)"
date:   2026-03-02 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw,insufficient-validation]
categories: [BugForge,daily,cheesy-does-it]
---

# Daily - Cheesy Does It (Repeat)

This is a repeat of the daily challenge from [January 26th, 2026](https://zwarts.dev/posts/2026/01/26/BugForge-Daily-Cheesy-Does-It/).

---

## Vulnerability Overview

A **Business Logic Flaw** exists in the `refund endpoint` where the server accepts arbitrary refund amounts without validating them against actual order values. By discovering the refund API endpoint and payload structure through client-side JavaScript analysis (`OrderTracking.js`), an attacker can submit a fraudulent refund request for any amount. The backend performs no cross-referencing against stored order records, allowing the request to be processed and the flag to be returned.

**Key Issues:**
- The refund API endpoint and payload are exposed through client-side JavaScript (`OrderTracking.js`)
- The backend accepts client-supplied refund amounts without validating them against the actual order total
- No authorization checks confirm the requesting user is the original purchaser of the associated order

---

## Vulnerabilities Covered

- Business Logic Flaw
- Insufficient Input Validation

---

## Classification

- **OWASP Top 10:** A04:2021 - Insecure Design
- **Vulnerability Type:** Business Logic Flaw / Insufficient Input Validation
- **Attack Surface:** Refund processing API endpoint
- **CWE:** CWE-840 - Business Logic Errors

---
