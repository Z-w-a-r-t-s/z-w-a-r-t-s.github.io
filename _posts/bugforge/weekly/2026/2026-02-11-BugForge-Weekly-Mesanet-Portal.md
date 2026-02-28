---
layout: post
title:  "BugForge - Weekly - Mesanet Portal"
date:   2026-02-11 09:00
image:  /images/bug-forge/bugforge-logo.png
tags:   [sqli,api-gateway-abuse,directory-enumeration]
categories: [BugForge,weekly,mesanet-portal]
---

# Weekly - Mesanet Portal
><br/><b>Vulnerabilities Covered:</b>
<br/>
SQL Injection (INSERT-based String Concatenation)
<br/>
API Gateway Abuse via Boolean Type Confusion
<br/>
Directory Enumeration
<br/>
<br/>
<b>Summary:</b>
<br/>
This walkthrough demonstrates a SQL injection vulnerability in a microservice-based portal that follows the API Gateway pattern. By analysing the application's frontend source code, a hardcoded internal service identifier was discovered. Boolean type confusion on the gateway's application ID parameter revealed a hidden internal rail service. Fuzzing the rail endpoints uncovered a create endpoint vulnerable to INSERT-based SQL injection via string concatenation. The injection was used to extract database tables, schema, and credentials from a config table. The extracted database admin credentials provided access to a database backup portal, which was used to download a portal database containing a one-time password required to access a dev console and retrieve the flag.
<br/>
<br/>
<b>Reference:</b>
<br/>
<a href="https://app.bugforge.io/">Bugforge.io</a>
<br/>
<br/>

## Solution

### Step 1 - Application Analysis & Directory Enumeration

Start by fuzzing for directories using the `Seclists/Discovery/Web-Content/common.txt` wordlist.

![Fuzzing](/images/bug-forge/weekly/mesanet-portal/sqli/fuzzing.png)
![Fuzzing Results](/images/bug-forge/weekly/mesanet-portal/sqli/fuzzing-results.png)

Two additional portals are discovered: a **dev portal** requiring a one-time password (OTP) and a **database admin portal** requiring credentials.

![Dev Portal UI](/images/bug-forge/weekly/mesanet-portal/sqli/dev-portal-ui.png)

![Database Admin Portal UI](/images/bug-forge/weekly/mesanet-portal/sqli/database-admin-ui.png)

---

### Step 2 - Authenticated Application Exploration

Log in with the provided credentials `operator:operator`. The application presents a dashboard with a notice board, secure email functionality, and the dev console discovered during enumeration.

![Dashboard UI](/images/bug-forge/weekly/mesanet-portal/sqli/dashboard-ui.png)

---

### Step 3 - Source Code Analysis & Gateway Discovery

Access the Nexus notice board and analyse the source code for `apps/nexus`. A hardcoded `APP_ID` is present along with a `POST` request to a gateway endpoint. The request payload contains a JSON object with `id`, `endpoint`, and a `data` object.

![Notice Board UI](/images/bug-forge/weekly/mesanet-portal/sqli/notice-board-ui.png)

![Nexus Source Code](/images/bug-forge/weekly/mesanet-portal/sqli/nexus-source-code.png)

The application follows the [API Gateway pattern](https://learn.microsoft.com/en-us/azure/architecture/microservices/design/gateway) a design approach where a single entry point sits between client applications and a collection of backend microservices, acting as a reverse proxy that routes incoming requests to the appropriate internal service. Instead of the client communicating directly with multiple backend services, all requests are sent to the gateway, which determines which internal service should handle the request based on parameters like a service identifier or route path.

![Gateway POST Request](/images/bug-forge/weekly/mesanet-portal/sqli/gateway-post-request.png)

---

### Step 4 - Gateway Parameter Manipulation & Type Confusion

Manipulate the gateway payload to understand its behaviour. Removing the `id` parameter returns the error `App ID and endpoint are required`. Changing the `id` value to an integer (`1`) returns `Unknown application ID`. However, setting the `id` to a boolean value (`true`) returns a different error: `Rail endpoint not found`.

![Rail Endpoint Not Found](/images/bug-forge/weekly/mesanet-portal/sqli/rail-endpoint-not-found.png)

This type confusion reveals the existence of an internal **rail** service. The boolean `true` bypasses the application ID lookup and routes directly to the rail service, exposing an entirely new attack surface.

---

### Step 5 - Fuzzing the Rail Service Endpoints

Fuzz the rail endpoint using the `Seclists/Discovery/Web-Content/common.txt` wordlist to enumerate available routes.

![Rail Endpoints](/images/bug-forge/weekly/mesanet-portal/sqli/rail-endpoints.png)

Several endpoints are discovered. Analyse each to understand the service functionality:

`announcements`returns a list of all announcements:

![Rail Announcements](/images/bug-forge/weekly/mesanet-portal/sqli/rail-announcements.png)

`current`returns the current announcement:

![Rail Current](/images/bug-forge/weekly/mesanet-portal/sqli/current-announcement.png)

`schedule`returns the schedule:

![Rail Schedule](/images/bug-forge/weekly/mesanet-portal/sqli/rail-schedule.png)

`status`returns the service status:

![Rail Status](/images/bug-forge/weekly/mesanet-portal/sqli/rail-status-endpoint.png)

---

### Step 6 - Identifying the SQL Injection Point

Send a request to the `create` endpoint with no data. The error message `type, message, timestamp, and priority are required` reveals the expected payload structure. Reference the data returned from `api/rail/announcements` to determine valid values for `type` and `priority`, then craft a valid creation request.

![Create Announcement](/images/bug-forge/weekly/mesanet-portal/sqli/create-announcement.png)

Inject a single quote (`'`) into the `message` field. The application returns a database error, indicating the `message` parameter is vulnerable to SQL injection.

![SQLI Error](/images/bug-forge/weekly/mesanet-portal/sqli/sqli-error.png)

---

### Step 7 - INSERT-based SQL Injection via String Concatenation

When the injection point is inside an `INSERT` or `UPDATE` statement, UNION-based extraction is not possible because there is no `SELECT` to append to. Instead, the `||` concatenation operator is used to embed a subquery result into the stored string value. The database evaluates the subquery at write time, stores the combined result, and when the application reflects it back in the response, the extracted data is visible.

Extract all tables from the database:

```sql
Zwarts' || (SELECT group_concat(name) FROM sqlite_master WHERE type='table') || '
```

![Database Tables](/images/bug-forge/weekly/mesanet-portal/sqli/database-tables.png)

Extract the column structure of the `config` table:

```sql
Zwarts' || (SELECT group_concat(sql) FROM sqlite_master WHERE type='table' AND name='config') || '
```

![Config Columns](/images/bug-forge/weekly/mesanet-portal/sqli/config-columns.png)

Dump the contents of the `config` table:

```sql
Zwarts' || (SELECT group_concat(key || ':' || value) FROM config) || '
```

![Config Data](/images/bug-forge/weekly/mesanet-portal/sqli/config-data.png)

The config table reveals database admin credentials: `dbadmin:Xen_Lambda_R4ilSyst3m_2024!Cr0ss1ng`

---

### Step 8 - Accessing the Database Admin Portal

Use the extracted credentials to log in to the database admin portal discovered during enumeration.

![Database Portal Login](/images/bug-forge/weekly/mesanet-portal/sqli/database-admin-portal.png)

The portal allows downloading database backups when provided with a database name. Attempting a generic name reveals an error stating the database name must be suffixed with `Db`.

![Database Name Error](/images/bug-forge/weekly/mesanet-portal/sqli/database-name-error.png)

---

### Step 9 - Fuzzing the Database Name

Generate a custom wordlist themed around the challenge context (Half-Life / Black Mesa references with the `Db` suffix) and fuzz the database name parameter:

```
mesaNetDb
blackMesaDb
anomalousMaterialsDb
lambdaCoreDb
sectorCDb
xenDb
nihilanthDb
gordonFreemanDb
barneyDb
eliVanceDb
alyxVanceDb
kleinerDb
magnussonDb
headcrabDb
vortigauntDb
combineDb
apertureDb
resonanceCascadeDb
hevSuitDb
gravityGunDb
crowbarDb
teleporterDb
borealisDb
ravenholmDb
city17Db
citadelDb
surfaceTensionDb
unforeseenConsequencesDb
powerUpDb
blastPitDb
apprehensionDb
questionableEthicsDb
interloperDb
endgameDb
freemanDb
lambdaDb
nexusDb
portalDb
mesaDb
researchDb
testChamberDb
siloDb
adminDb
securityDb
personnelDb
facilityDb
containmentDb
hazardCourseDb
decayDb
opposingForceDb
blueshiftDb
sectorGDb
sectorEDb
sectorBDb
sectorADb
sectorDDb
sectorFDb
level3Db
subLevelDb
canalDb
wastelandDb
highwayDb
novaProsektDb
sandtrapsDb
antiMassDb
specimenDb
cascadeDb
xenCrystalDb
boundaryDb
crossfireDb
dataCoreDb
mainframeDb
mesaLabDb
mesaSecurityDb
mesaAdminDb
mesaPersonnelDb
mesaAccessDb
mesaPortalDb
mesaCoreDb
mesaNetAccessDb
accessPortalDb
portalAccessDb
netAccessDb
mesaNetCoreDb
mesaNetAdminDb
mesaNetSecurityDb
```

The fuzzing returns a hit on `portalDb`.

---

### Step 10 - Extracting the OTP & Retrieving the Flag

Navigate to the dev console and trigger the OTP generation. Return to the database admin portal, download the `portalDb` backup, and extract the OTP from the database contents.

![Read OTP](/images/bug-forge/weekly/mesanet-portal/sqli/read-otp.png)

Enter the OTP in the dev console to gain access and retrieve the flag.

![Flag](/images/bug-forge/weekly/mesanet-portal/sqli/flag.png)

---

### Impact

- Full database schema enumeration and data extraction via SQL injection
- Exposure of database admin credentials stored in plaintext configuration
- Access to database backup functionality enabling offline analysis of all application data
- Bypass of OTP-protected dev console through database credential chain
- Discovery of hidden internal microservices via API gateway type confusion
- Potential lateral movement to other internal services routed through the gateway

---

### Vulnerability Classification

- **OWASP Top 10:** A03:2021 - Injection
- **Vulnerability Type:** INSERT-based SQL Injection via String Concatenation
- **Attack Surface:** API Gateway rail service `create` endpoint, `message` parameter
- **Database:** SQLite
- **CWE:**
  - CWE-89 - Improper Neutralization of Special Elements used in an SQL Command (SQL Injection)
  - CWE-843 - Access of Resource Using Incompatible Type (Type Confusion)
  - CWE-256 - Plaintext Storage of a Password

---

### Root Cause

The rail service's `create` endpoint constructs an `INSERT` statement by directly concatenating user-supplied input from the `message` parameter into the SQL query without parameterization. This allows an attacker to inject arbitrary SQL subqueries that are evaluated at write time and reflected in subsequent read operations. The vulnerability chain is compounded by the API gateway's weak type handling, which allows a boolean value to bypass application ID validation and expose the internal rail service. Additionally, storing database credentials in plaintext within a config table and exposing a database backup portal without rate limiting or additional authentication enables the full exploitation chain from SQL injection to flag retrieval.

---

### Remediation

- Use parameterized queries or prepared statements for all database operations, including INSERT and UPDATE statements
- Implement strict type validation on the API gateway to reject unexpected parameter types for the application ID
- Remove hardcoded service identifiers from client-side source code
- Store sensitive credentials using a secrets management solution rather than in database configuration tables
- Apply network-level access controls to restrict direct access to internal microservices
- Implement rate limiting and additional authentication on the database backup portal
- Conduct regular security code reviews focusing on data flow through the API gateway to backend services

---
