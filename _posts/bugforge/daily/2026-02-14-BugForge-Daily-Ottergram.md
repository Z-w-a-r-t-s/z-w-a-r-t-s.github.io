---
layout: post
title:  "BugForge - Daily - Ottergram"
date:   2026-02-14 19:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [broken-access-control,missing-authentication]
categories: [BugForge,daily,ottergram]
---

# Daily - Ottergram
><br/><b>Vulnerabilities Covered:</b>
<br/>
Broken Access Control - Missing Authentication on Comment Update Endpoint
<br/>
<br/>
<b>Summary:</b>
<br/>
The Ottergram application contains a **`Broken Access Control`** vulnerability in its comment update endpoint. After systematically analyzing the application's registration, profile management, and JWT token for privilege escalation opportunities, the comment editing functionality was identified as the attack surface. The `PUT` request to update comments correctly enforces authorization when a valid JWT token is present, returning a `403 Forbidden` when attempting to modify another user's comment. However, removing the `Authorization` header entirely bypasses all access control checks, allowing an unauthenticated request to successfully modify any user's comment. This demonstrates a critical flaw where the server only validates permissions when an authorization token is provided but fails to enforce authentication as a prerequisite for the operation.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis
Analyze the application's attack surface by examining key areas for access control weaknesses. Start with the registration request to check if the system accepts user-controlled role parameters, then move to profile management. Intercepting the profile update request reveals no user IDs or usernames in the payload, and the JWT token contains no role information, ruling out privilege escalation through these vectors.

---

### Step 2 - Identify the Comment Edit Functionality
Post a comment on the platform and observe that authenticated users can edit their own comments. 

![Edit comment UI](/images/bug-forge/daily/ottergram/comments/edit-comment-ui.png)

Attempt to edit another user's comment by modifying the comment ID in the `PUT` request while keeping your own JWT token in the `Authorization` header.

![PUT - Update another user's comments failed.](/images/bug-forge/daily/ottergram/comments/put-comments-403.png)

The server correctly returns a `403 Forbidden` response, indicating that authorization checks are in place when a valid token is provided.

---

### Step 3 - Remove the Authorization Header
Before exploring JWT-based attacks, test whether authentication is required at all by removing the `Authorization` header entirely from the `PUT` request to update another user's comment. Replay the request without any authentication credentials.

![Successful comment update](/images/bug-forge/daily/ottergram/comments/success-put-comments.png)

The server returns a success response, confirming that the comment was updated without any authentication. The endpoint only enforces authorization when a token is present but does not require authentication as a baseline check.

---

### Step 4 - Verify Exploitation
Use `GET /api/posts/1/comments` to retrieve all comments from post 1 and confirm that the target comment was successfully modified, revealing the challenge flag.

![Flag](/images/bug-forge/daily/ottergram/comments/flag.png)

---

### Impact
- Any unauthenticated user can modify arbitrary comments by removing the Authorization header
- Complete bypass of access control on the comment update endpoint
- Potential for defacement, misinformation, or manipulation of user-generated content
- Undermines trust in the platform's content integrity and moderation system
- If similar patterns exist across other endpoints, unauthenticated users could perform additional unauthorized actions

---

### Vulnerability Classification
- **OWASP Top 10:** A01:2021 - Broken Access Control
- **Vulnerability Type:** Missing Authentication on Protected Resource
- **Attack Surface:** Comment update endpoint (`PUT /api/posts/:id/comments/:commentId`)
- **CWE:** CWE-306 - Missing Authentication for Critical Function
- **CWE:** CWE-862 - Missing Authorization

---

### Root Cause
The comment update endpoint implements authorization logic that checks whether the authenticated user owns the comment, but this check is only triggered when an `Authorization` header is present in the request. When the header is absent, the server skips the authorization check entirely rather than rejecting the request as unauthenticated. The application fails to enforce authentication as a mandatory prerequisite before processing the request, allowing unauthenticated users to bypass all access controls by simply omitting their credentials.

---

### Remediation
- Enforce authentication as a mandatory check before any authorization logic; reject requests without valid credentials immediately with a `401 Unauthorized` response
- Use authentication middleware that runs before route handlers to ensure all protected endpoints require a valid session or token
- Never treat the absence of an authorization token as an implicit bypass; default to deny when credentials are missing
- Apply the principle of defense in depth by layering authentication and authorization checks independently
- Conduct security testing that includes removing or omitting authentication headers to identify endpoints with missing authentication enforcement
- Implement integration tests that verify unauthenticated requests are rejected across all protected endpoints

---
