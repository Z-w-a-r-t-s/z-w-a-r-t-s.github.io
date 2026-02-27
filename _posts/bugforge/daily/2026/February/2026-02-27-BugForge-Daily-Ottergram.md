---
layout: post
title:  "BugForge - Daily - Ottergram (Repeat)"
date:   2026-02-27 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,parameter-tampering,insecure-design]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram (Repeat)

This is a repeat of the daily challenge from [January 2nd, 2026](https://zwarts.dev/posts/2026/01/02/BugForge-Daily-Shady-Oaks-Finance/).

---

## Vulnerability Overview

A **Broken Access Control** vulnerability caused by **Insecure Design** exists in the `application's upgrade feature`. Rather than handling role elevation logic server-side, the application accepts a `role` parameter directly from the client during the upgrade process. By intercepting and modifying the upgrade request to change the role value from a standard user to `administrator`, an attacker can escalate privileges and gain unauthorized access to administrative functionality. This demonstrates a **fundamental design flaw** where authorization decisions are delegated to the client rather than being enforced by the backend.

**Key Issues:**
- The application accepts a client-supplied `role` parameter in the upgrade request body, trusting the client to specify its own role
- No server-side validation or authorization checks are performed on the supplied role value before applying it to the user's account
- Authorization decisions are delegated to the client rather than enforced by the backend, enabling any authenticated user to escalate to administrator

---

## Vulnerabilities Covered

- Broken Access Control
- Parameter Tampering
- Insecure Design / Privilege Escalation

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Parameter Tampering / Insecure Design / Privilege Escalation
- **Attack Surface:** Upgrade API endpoint
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
