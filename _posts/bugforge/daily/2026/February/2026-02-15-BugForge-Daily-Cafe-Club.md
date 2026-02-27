---
layout: post
title:  "BugForge - Daily - Cafe Club (Repeat)"
date:   2026-02-15 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [business-logic-flaw]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club (Repeat)

This is a repeat of the daily challenge from [January 4th, 2026](https://zwarts.dev/posts/2026/01/04/BugForge-Daily-Cafe-Club/).

---

## Vulnerability Overview

The CafeClub application contains a business logic vulnerability in the checkout process where the server fails to validate the `points_to_use` parameter against the user's actual point balance. By intercepting the checkout request and modifying the points value to exceed the available balance, an attacker can redeem more points than they possess, effectively obtaining products for free. The server blindly trusts the client-supplied value, resulting in negative point balances and unauthorized discounts.

**Key Issues:**
- The checkout endpoint accepts a client-supplied `points_to_use` parameter without server-side validation
- The backend does not verify whether the user's actual point balance covers the requested redemption amount
- Client-side UI restrictions (disabling the points option when balance is zero) are the only control, which is trivially bypassed via request interception
- Attackers can apply arbitrary discounts by inflating the `points_to_use` value, obtaining products at zero cost

This represents a business logic flaw where insufficient server-side validation of business rules enables financial exploitation of the checkout process.

---

## Vulnerabilities Covered

- **Business Logic Flaw / Insufficient Input Validation** - The checkout API trusts client-supplied point redemption values without validating against the user's actual balance, allowing unauthorized discounts and free product acquisition

---

## Classification

- **OWASP Top 10:** A04:2021 - Insecure Design
- **CWE:** CWE-20 - Improper Input Validation

---
