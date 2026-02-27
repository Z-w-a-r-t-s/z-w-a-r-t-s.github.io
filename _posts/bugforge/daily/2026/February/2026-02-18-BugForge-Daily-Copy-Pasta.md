---
layout: post
title:  "BugForge - Daily - Copy Pasta (Repeat)"
date:   2026-02-18 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [idor]
categories: [BugForge,daily,copy-pasta]
---

# Daily - Copy Pasta (Repeat)

This is a repeat of the daily challenge from [January 14th, 2026](https://zwarts.dev/posts/2026/01/14/BugForge-Daily-Copy-Pasta/).

---

## Vulnerability Overview

This challenge demonstrates an Insecure Direct Object Reference (IDOR) vulnerability in the delete functionality of a code snippet application. While the application correctly restricted unauthorized updates to other users' snippets, the delete endpoint accepted a snippet ID as a query parameter without validating ownership. By enumerating snippet IDs through the delete request, it was possible to delete snippets belonging to other users, ultimately retrieving the flag through unauthorized deletion.

**Key Issues:**
- The delete endpoint passes the snippet ID as a query parameter with no server-side ownership check
- Authorization controls were applied inconsistently, update requests were protected but delete requests were not
- Sequential or enumerable snippet IDs allowed straightforward iteration to discover and target other users' resources

---

## Vulnerabilities Covered

- IDOR (Insecure Direct Object Reference)

---

## Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Insecure Direct Object Reference (IDOR)
- **Attack Surface:** Snippet delete endpoint with user-controlled query parameter
- **CWE:** CWE-639 - Authorization Bypass Through User-Controlled Key

---
