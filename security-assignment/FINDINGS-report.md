# Web Application Security Assessment Report

**Target:** OWASP Juice Shop — `http://localhost:3000` (local Docker instance)
**Author:** [Your Name] — [Course / Student ID]
**Date:** 2026-10-05
**Version:** 1.0 — Confidential (Coursework)

---

## 2. Authorization & Scope

- **Target:** `http://localhost:3000`, OWASP Juice Shop (`bkimminich/juice-shop`) running in a
  self-hosted Docker container.
- **Authorization basis:** The application is a deliberately vulnerable app published by OWASP
  for security training, and the instance + host are owned by the tester. No third-party
  system was touched. **(For your own submission, state your authorization here exactly like
  this — a real public site would require written owner permission.)**
- **In scope:** All application endpoints on `localhost:3000`.
- **Out of scope:** Denial-of-service, host OS, Docker daemon.
- **Testing window:** 2026-10-05.
- **Method:** Black-box, manual testing with `curl`, aligned to OWASP WSTG / Top 10 (2021).

---

## 3. Executive Summary

Testing identified **7 findings: 2 Critical, 1 High, 3 Medium, 1 Low.** The application is
critically vulnerable to **SQL injection**, allowing both authentication bypass (login as
administrator without credentials) and full exfiltration of the user database, including every
user's email and password hash. Password hashes use **unsalted MD5**, making them trivially
crackable. Supporting weaknesses (verbose errors, missing headers, permissive CORS, sensitive
data in tokens) compound the exposure.

**Overall risk rating: Critical.** An unauthenticated attacker can achieve full administrative
compromise and mass credential theft.

---

## 4. Methodology

Manual black-box testing following the OWASP Web Security Testing Guide, assessing against the
OWASP Top 10 (2021): reconnaissance → configuration/headers → authentication → session/token
handling → authorization → input validation (injection) → error handling. Severity is scored
with **CVSS v3.1**.

**Tools:** `curl`, browser dev tools, base64/JWT decoding.

---

## 5. Findings Summary

| # | Finding | OWASP Top 10 | Severity | CVSS v3.1 |
|---|---|---|---|---|
| F-01 | SQL injection — authentication bypass (admin login) | A03: Injection | **Critical** | 9.8 |
| F-02 | SQL injection — UNION-based full DB exfiltration | A03: Injection | **Critical** | 9.1 |
| F-03 | Weak password storage — unsalted MD5 | A02: Cryptographic Failures | **High** | 7.5 |
| F-04 | Sensitive data exposure in JWT payload | A02: Cryptographic Failures | **Medium** | 6.5 |
| F-05 | Verbose SQL error messages (info leakage) | A05: Security Misconfiguration | **Medium** | 5.3 |
| F-06 | Overly permissive CORS (`Access-Control-Allow-Origin: *`) | A05: Security Misconfiguration | **Medium** | 5.3 |
| F-07 | Missing HTTP security headers (CSP, HSTS, Referrer-Policy, Permissions-Policy) | A05: Security Misconfiguration | **Low** | 3.7 |

---

## 6. Detailed Findings

### F-01 — SQL Injection: Authentication Bypass

- **Severity:** Critical | **CVSS v3.1:** 9.8 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
- **OWASP:** A03:2021 Injection
- **Affected:** `POST /rest/user/login`, `email` parameter

**Description**
The `email` field in the login request is concatenated directly into a SQL query. Supplying
`' OR 1=1--` makes the WHERE clause always true, returning the first user (the administrator)
and issuing a valid authentication token — no password required.

**Steps to Reproduce**
```
POST /rest/user/login
Content-Type: application/json

{"email":"' OR 1=1--","password":"x"}
```
**Result:** HTTP 200 with a valid JWT for `admin@juice-sh.op` (`"role":"admin"`). An invalid
normal login returns `"Invalid email or password."` — confirming the bypass.

**Impact**
Complete authentication bypass and administrative account takeover by an unauthenticated
attacker.

**Remediation**
Use parameterized queries / prepared statements (or the ORM's safe query builder) for all user
input. Never concatenate input into SQL. Add input validation and a WAF as defense-in-depth.

---

### F-02 — SQL Injection: UNION-Based Database Exfiltration

- **Severity:** Critical | **CVSS v3.1:** 9.1 (AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:L/A:N)
- **OWASP:** A03:2021 Injection
- **Affected:** `GET /rest/products/search`, `q` parameter

**Description**
The product-search `q` parameter is injectable. A UNION SELECT appends attacker-controlled
rows from arbitrary tables to the results, enabling extraction of the entire `Users` table.

**Steps to Reproduce**
```
GET /rest/products/search?q=qwert')) UNION SELECT id,email,password,role,5,6,7,8,9 FROM Users--
```
**Result (excerpt):**
```json
{"status":"success","data":[
  {"id":1,"name":"admin@juice-sh.op","description":"0192023a7bbd73250516f069df18b500","price":"admin"},
  {"id":2,"name":"jim@juice-sh.op","description":"e541ca7ecf72b8d1286474fc613e5e45","price":"customer"},
  {"id":3,"name":"bender@juice-sh.op","description":"0c36e517e3fa95aabf1bbffc6744a4ef","price":"customer"}
]}
```
Emails map to `name`, password hashes to `description`, roles to `price`.

**Impact**
Mass theft of all user credentials and arbitrary read of the database.

**Remediation**
Parameterized queries; validate/whitelist the `q` parameter; least-privilege DB account;
generic error handling.

---

### F-03 — Weak Password Storage (Unsalted MD5)

- **Severity:** High | **CVSS v3.1:** 7.5
- **OWASP:** A02:2021 Cryptographic Failures
- **Affected:** User credential store

**Description**
Password hashes recovered in F-02 are 32-hex-character **unsalted MD5** digests (e.g. the admin
hash `0192023a7bbd73250516f069df18b500` is MD5 of a weak password). MD5 is fast and unsalted,
so hashes fall instantly to rainbow tables / wordlists.

**Impact**
Once hashes are exfiltrated, plaintext passwords are trivially recovered, enabling account
takeover and credential-stuffing against other services.

**Remediation**
Store passwords with a slow, salted algorithm: **bcrypt, scrypt, or Argon2id**. Enforce a strong
password policy and force a reset after any exposure.

---

### F-04 — Sensitive Data Exposure in JWT Payload

- **Severity:** Medium | **CVSS v3.1:** 6.5
- **OWASP:** A02:2021 Cryptographic Failures
- **Affected:** Authentication token (`/rest/user/login` response)

**Description**
The issued JWT payload embeds the full user record — including `email`, `password` (hash),
`role`, and internal fields. JWT payloads are only base64-encoded, not encrypted, so anyone
holding the token can read them.

**Evidence (decoded payload):** `"email":"admin@juice-sh.op"`, `"password":"0192023a7bbd...",`
`"role":"admin"`.

**Impact**
Leaks the password hash and internal data to the client and anywhere the token is logged/cached.

**Remediation**
Put only a minimal, non-sensitive subject identifier in the token. Never include password hashes
or full records. Keep token lifetimes short.

---

### F-05 — Verbose SQL Error Messages

- **Severity:** Medium | **CVSS v3.1:** 5.3
- **OWASP:** A05:2021 Security Misconfiguration
- **Affected:** Application error handler

**Description**
A malformed query (`q=test'`) returns a full error page disclosing the database engine and
syntax detail: `Error: SQLITE_ERROR: near "'%'": syntax error`, confirming SQLite and aiding
injection tuning.

**Impact**
Information leakage that accelerates exploitation (F-01/F-02).

**Remediation**
Return generic error pages; log details server-side only; disable stack traces in production.

---

### F-06 — Overly Permissive CORS Policy

- **Severity:** Medium | **CVSS v3.1:** 5.3
- **OWASP:** A05:2021 Security Misconfiguration
- **Affected:** Global HTTP responses (`Access-Control-Allow-Origin: *`)

**Description**
All responses send `Access-Control-Allow-Origin: *`, allowing any origin to make cross-origin
requests to the API.

**Impact**
Any malicious website can call the API in a victim's browser context, increasing the blast
radius of other flaws.

**Remediation**
Restrict `Access-Control-Allow-Origin` to an explicit allowlist of trusted origins; avoid the
wildcard, especially for authenticated endpoints.

---

### F-07 — Missing HTTP Security Headers

- **Severity:** Low | **CVSS v3.1:** 3.7
- **OWASP:** A05:2021 Security Misconfiguration
- **Affected:** All HTTP responses

**Description**
The following headers are absent: **Content-Security-Policy**, **Strict-Transport-Security**,
**Referrer-Policy**, **Permissions-Policy**. (`X-Frame-Options` and `X-Content-Type-Options`
are present.)

**Impact**
Weakens browser-side defenses against XSS/clickjackjacking, referrer leakage, and protocol
downgrade.

**Remediation**
Add `Content-Security-Policy` (restrictive), `Strict-Transport-Security` (HTTPS), `Referrer-Policy:
strict-origin-when-cross-origin`, and a `Permissions-Policy`.

---

## 7. Remediation Roadmap

| Priority | Finding(s) | Action | Effort |
|---|---|---|---|
| 1 | F-01, F-02 | Replace all raw SQL with parameterized queries | Medium |
| 2 | F-03 | Migrate password hashing to Argon2id/bcrypt + force reset | Medium |
| 3 | F-04, F-05 | Minimize JWT claims; generic error handling | Small |
| 4 | F-06, F-07 | Lock down CORS; add security headers | Small |

---

## 8. Appendix — WSTG Coverage

Configuration/headers, authentication, session/token handling, and input-validation (injection)
testing were performed. Not covered in this pass (recommended next): stored/DOM XSS, broken
access control / IDOR across user resources, file-upload handling, and business-logic abuse.
