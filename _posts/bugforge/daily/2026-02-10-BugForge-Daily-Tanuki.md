---
layout: post
title:  "BugForge - Daily - Tanuki (Repeat)"
date:   2026-02-10 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [mass-assignment]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki (Repeat)

This is a repeat of the daily challenge from
[December 30th, 2025](https://zwarts.dev/posts/2025/12/30/BugForge-Daily-Tanuki/).

---

## Vulnerability Overview

This vulnerability is a **mass assignment-driven privilege escalation** where the application trusts client-supplied input during user registration and allows sensitive attributes, such as `user_role`, to be set directly by the user. By manipulating the registration request and changing the role from a standard user to an administrator, an attacker can create an account with elevated privileges. The absence of server-side allowlisting and role enforcement results in a complete breakdown of authorization controls and unrestricted access to administrative functionality.

**Key Issues:**
- The application allows the `user_role` parameter to be included in the client-side registration payload
- No server-side allowlist or enforcement restricts which fields may be modified during account creation
- The backend implicitly trusts client-supplied input and assigns roles based on user-controlled values
- Attackers can escalate privileges by modifying the role value from `user` to `admin` during registration

This represents a classic mass assignment vulnerability where insufficient input filtering enables vertical privilege escalation.

---

## Vulnerabilities Covered

- **Mass Assignment** - Exploitation of unprotected object attribute binding to set sensitive fields such as `user_role` during user registration
- **Broken Access Control** - Missing server-side enforcement allowing vertical privilege escalation to administrative roles

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **CWE:** CWE-915 - Improperly Controlled Modification of Dynamically-Determined Object Attributes

---
