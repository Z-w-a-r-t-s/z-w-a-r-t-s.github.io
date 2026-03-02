---
layout: post
title:  "BugForge - Daily - Cafe Club"
date:   2026-03-01 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [file-inclusion,path-traversal]
categories: [BugForge,daily,cafe-club]
---

# Daily - Cafe Club
><br/><b>Vulnerabilities Covered:</b>
<br/>
File Inclusion - Path Traversal<br/>
<br/>
<b>Summary:</b>
<br/>
The Cafe Club application contains a **`Path Traversal`** vulnerability in its `product image loading` endpoint. The endpoint accepts a user-controlled file path with no sanitization, allowing directory traversal sequences to escape the intended directory. By supplying `../../../flag.txt`, the server reads and returns arbitrary files from the filesystem without restriction.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Reconnaissance

After registering our account, notice all the requests to load product images, next we'll attempt file inclusions

![Requests - Loading item images](/images/bug-forge/daily/cafe-club/file-inclusion/file-requests.png)

Intercepting these requests reveals that the image endpoint accepts a file path as a parameter. The parameter is passed directly to the server's file system without any sanitization. Modify the file path to use directory traversal sequences to escape the web root and target sensitive files on the server.

Supply `../../../flag.txt` as the file path value in the image request.

![Flag](/images/bug-forge/daily/cafe-club/file-inclusion/flag.png)

The server resolves the traversal sequence and returns the contents of `flag.txt`, confirming unrestricted access to the file system.

---

### Impact

- Any user can read arbitrary files from the server's filesystem by manipulating the image path parameter
- Sensitive files such as configuration files, credentials, environment variables, and application source code are exposed
- An attacker could read `/etc/passwd`, `.env` files, private keys, or other secrets accessible to the server process
- No authentication bypass is required; the vulnerable endpoint is reachable by any user who can interact with the application
- Full filesystem disclosure is possible up to the privilege level of the server process

---

### Vulnerability Classification

- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Path Traversal / Local File Inclusion
- **Attack Surface:** Product image loading endpoint, user-controlled file path parameter
- **CWE:** CWE-22 - Improper Limitation of a Pathname to a Restricted Directory (Path Traversal)
- **CWE:** CWE-73 - External Control of File Name or Path

---

### Root Cause

The product image endpoint constructs a file path by directly concatenating a user-supplied value with the server's base directory, without performing any validation, sanitization, or canonicalization. Directory traversal sequences (`../`) are not stripped or blocked, allowing the resolved path to escape the intended image directory entirely. The server then reads and returns the contents of the resolved path, regardless of its location within the filesystem. The absence of path normalization and boundary enforcement is the fundamental root cause.

---

### Remediation

- Validate and sanitize all user-supplied file paths before use; reject any input containing traversal sequences such as `../` or encoded equivalents
- Resolve the canonical path of the requested file and verify it falls within the intended base directory before reading it; return `400 Bad Request` or `403 Forbidden` if the resolved path escapes the allowed directory
- Use an allowlist of permitted filenames or file extensions rather than accepting arbitrary user input as a file path
- Serve static assets through a dedicated web server or CDN rather than resolving them dynamically via application logic
- Store files using opaque, non-guessable identifiers (e.g., UUIDs) mapped server-side to actual paths, so users never supply a raw file path
- Apply the principle of least privilege to the server process so that even if traversal occurs, the process cannot read sensitive system files

---
