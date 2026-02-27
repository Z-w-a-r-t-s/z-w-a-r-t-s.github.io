---
layout: post
title:  "BugForge - Daily - Tanuki (Repeat)"
date:   2026-02-17 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [xxe]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki (Repeat)

This is a repeat of the daily challenge from [January 6th, 2026](https://zwarts.dev/posts/2026/01/06/BugForge-Daily-Tanuki/).

---

## Vulnerability Overview

This challenge demonstrates an XML External Entity (XXE) vulnerability exploitable through XInclude processing in the Import Deck functionality. While traditional DTD-based XXE payloads were blocked by the application, the XML parser retained support for XInclude processing. By declaring the XInclude namespace on the root element and using `xi:include` to reference local files, arbitrary file disclosure was achieved. This highlights that disabling external entity expansion alone is insufficient-XInclude processing must also be disabled to fully mitigate XXE-style attacks.

**Key Issues:**
- The backend XML parser disabled traditional external entity expansion but left XInclude processing enabled
- User-supplied XML documents containing XInclude directives are parsed and processed by the server
- The incomplete mitigation created a false sense of security while leaving an alternative exploitation path available

---

## Vulnerabilities Covered

- XXE (XInclude)

---

## Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** XML External Entity (XXE) via XInclude
- **Attack Surface:** File upload and server-side XML parsing functionality
- **CWE:** CWE-611 - Improper Restriction of XML External Entity Reference

---
