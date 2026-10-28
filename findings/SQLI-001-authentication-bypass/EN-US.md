# SQLI-001 — SQL Injection: Authentication Bypass

## Identification
- Finding ID: SQLI-001
- Date: 2026-10-28
- Target: OWASP Juice Shop (isolated Docker laboratory)
- Endpoint: POST /rest/user/login
- Category: SQL Injection / Authentication Bypass
- CWE: CWE-89
- OWASP WSTG: WSTG-INPV-05
- Status: Confirmed
- Preliminary severity: Critical (CVSS pending)

## Description
The authentication endpoint improperly handles SQL metacharacters
in the email parameter.

A crafted input altered the authentication query logic, allowing
the application to issue an authentication token without the
corresponding account password.

## Evidence

### Test 1 — Invalid credentials
HTTP response: 401 Unauthorized
Result: Authentication rejected.

### Test 2 — Single quote in email
HTTP response: 500 Internal Server Error
Result: Database-related stack trace disclosed SQLite/Sequelize paths.

### Test 3 — SQL Injection
Payload pattern: ' OR 1=1--
HTTP response: 200 OK

Observed authentication response:
- token: present
- bid: present
- umail: admin@juice-sh.op

Result: Authentication token issued despite an invalid password.

No authentication tokens or session secrets are stored in this report.

## Impact
An unauthenticated attacker may bypass password verification and
obtain an authenticated session for another account.

The laboratory test demonstrated token issuance associated with
the administrative account.

Administrative operations were not tested.

## Root Cause
Suspected unsafe construction of SQL queries using user-controlled
input. Source-level confirmation remains pending.

## Remediation
1. Use parameterized queries and safe ORM query methods.
2. Never concatenate user input into SQL statements.
3. Verify passwords independently using secure password hashing.
4. Return generic authentication errors.
5. Disable detailed stack traces in production responses.
6. Add regression tests for SQL injection in authentication.

## Validation
- Vulnerability reproduced in the authorized local laboratory.
- Authentication bypass confirmed.
- Remediation and retesting pending.

## Evidence Handling
Only sanitized HTTP responses and screenshots may be published.
Never publish JWTs, session cookies, passwords, or private keys.

## Confirmed Root Cause — Source Code Analysis

File: routes/login.ts
Line: 34

The application executes a raw SQL query through
models.sequelize.query().

The user-controlled req.body.email parameter is
directly interpolated into the SQL statement.

A crafted single quote and SQL comment sequence
can remove the password verification condition
from the effective query.

Source code analysis confirms the root cause
of SQLI-001.

Remediation requires parameterized queries
instead of direct interpolation of user input.

## Remediation Retest — Evidence

### Environment

- Application: OWASP Juice Shop
- Original container: juice-shop-lab
- Patched container: juice-shop-fixed
- Remediation: bound parameters in the authentication SQL query
- Patched file: build/routes/login.js

### Test 1 — SQL Injection After Remediation

**Input:** previously confirmed authentication bypass payload.

**Observed result:**
- HTTP: 401 Unauthorized
- Token issued: no
- Authentication granted: no

**Conclusion:** PASS. The tested payload did not grant access.

### Test 2 — Legitimate Account Registration

**Observed result:**
- HTTP: 201 Created

**Conclusion:** PASS. Test account registration remained functional.

### Test 3 — Legitimate Authentication

**Observed result:**
- HTTP: 200 OK
- Token issued: yes
- Authentication granted: yes

**Conclusion:** PASS. Legitimate authentication remained functional.

### Final Assessment

The remediation passed the executed tests, prevented the
previously demonstrated exploit, and preserved legitimate
authentication functionality.

These results do not establish that every possible SQL
injection variant has been eliminated.

No credentials, session cookies, or authentication tokens
are included in this documentation.

## Vulnerable Code and Remediation

### Vulnerable code

```typescript
const sql = `SELECT * FROM Users WHERE email = '${req.body.email}'`;
```

User-controlled input is directly interpolated into SQL.

### Remediated code

```typescript
const sql = "SELECT * FROM Users WHERE email = $email";
const options = { bind: { email: req.body.email } };
```

Bound parameters separate user data from SQL syntax.

### Retest

- SQL Injection: HTTP 401, no token issued.
- Legitimate login: HTTP 200.

These excerpts are simplified to illustrate the root cause and remediation technique.
