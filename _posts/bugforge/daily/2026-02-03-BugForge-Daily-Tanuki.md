---
layout: post
title:  "BugForge - Daily - Tanuki"
date:   2026-02-03 20:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [ssrf]
categories: [BugForge,daily,tanuki]
---

# Daily - Tanuki
><br/><b>Vulnerabilities Covered:</b>
<br/>
Server-Side Request Forgery (SSRF)
<br/>
<br/>
<b>Summary:</b>
<br/>
This challenge demonstrates a Server-Side Request Forgery (SSRF) vulnerability in the Tanuki application's leaderboard functionality. The application exposes an `/api/fetch` endpoint that accepts a URL parameter to retrieve data from internal services, specifically using `http://localhost:3000/leaderboard` to load leaderboard data. By intercepting and modifying this request to target `http://localhost:3000/admin`, an attacker can bypass external access controls and retrieve sensitive administrative data directly from the internal network. This highlights a critical flaw where user-controlled input is passed directly to server-side HTTP requests without proper validation or URL allowlisting, enabling access to restricted internal endpoints that would otherwise be blocked from external access.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
After logging in, observe the new Leader Board feature on the dashboard. Click on View Rankings to access the leaderboard interface. Use browser developer tools or a proxy to analyze the network requests made when loading this feature.

![New Feature](/images/bug-forge/daily/Tanuki/ssrf/new-feature-ui.png)

![Dashboard](/images/bug-forge/daily/Tanuki/ssrf/leaderboard-ui.png)

---

### Step 2 - Analyze Leader Board Functionality
Examine the `/api/fetch` request that powers the leaderboard. Notice that the application sends a URL parameter containing `http://localhost:3000/leaderboard` to fetch data from an internal service. This pattern indicates the server is making HTTP requests on behalf of the client, a classic indicator of potential SSRF vulnerability.

![Fetch Request](/images/bug-forge/daily/Tanuki/ssrf/fetch-request.png)

---

### Step 3 - Exploiting the SSRF Vulnerability
Intercept the `/api/fetch` request and modify the URL parameter from `http://localhost:3000/leaderboard` to `http://localhost:3000/admin`. The server processes this request and fetches the admin endpoint internally, bypassing any external access restrictions. The response contains the flag, confirming successful exploitation of the SSRF vulnerability to access restricted internal resources.

![Flag](/images/bug-forge/daily/Tanuki/ssrf/flag.png)

---

### Impact
- Unauthorized access to internal services and administrative endpoints
- Bypass of network segmentation and firewall rules protecting internal resources
- Potential for reading sensitive configuration files or credentials from internal services
- Ability to scan and enumerate internal network infrastructure
- Risk of pivoting to other internal systems through the vulnerable server
- Potential for data exfiltration from backend databases or APIs
- Complete bypass of intended access control mechanisms for protected endpoints

---

### Vulnerability Classification
- **OWASP Top 10:** A10:2021 - Server-Side Request Forgery (SSRF)
- **Vulnerability Type:** Server-Side Request Forgery (SSRF)
- **Attack Surface:** Fetch API endpoint (`/api/fetch`)
- **CWE:** CWE-918 - Server-Side Request Forgery (SSRF)

---

### Root Cause
The application implements a server-side fetch mechanism that accepts user-controlled URLs without proper validation or restrictions. The `/api/fetch` endpoint directly passes the client-supplied URL to an internal HTTP client, allowing the server to make requests to arbitrary destinations including localhost and internal network addresses. No allowlist of permitted domains or URL validation is implemented, and the server blindly trusts the user-supplied input. This enables attackers to leverage the server as a proxy to access internal resources that are otherwise inaccessible from external networks.

---

### Remediation
- Implement a strict allowlist of permitted domains and endpoints for the fetch functionality
- Validate and sanitize all user-supplied URLs before processing
- Block requests to localhost, 127.0.0.1, internal IP ranges (10.x.x.x, 172.16.x.x, 192.168.x.x), and cloud metadata endpoints
- Use a URL parser to extract and validate the hostname before making requests
- Implement network-level segmentation to prevent the application server from accessing sensitive internal services
- Consider using a dedicated proxy service with strict egress filtering for external URL fetching
- Disable unnecessary URL schemes (file://, gopher://, dict://) that could be abused
- Add logging and monitoring for unusual fetch patterns or requests to internal addresses
- Apply the principle of least privilege to limit what internal resources the application can access

---
