# CRYPTO-001 — Weak Password Hashing (MD5)

**Application:** OWASP Juice Shop
**Environment:** Authorized Docker laboratory
**Status:** Confirmed — Remediation pending
**Classification:** CWE-916

## Description

The application uses MD5 to hash user passwords without an individual salt.

## Technical evidence

Source: `/juice-shop/models/user.ts`

```typescript
this.setDataValue('password', security.hash(clearTextPassword))
```

Source: `/juice-shop/lib/insecurity.ts`

```typescript
export const hash = (data: string) =>
  crypto.createHash('md5').update(data).digest('hex')
```

## Impact

MD5 is unsuitable for password storage because it is computationally fast and does not provide an individual salt.

Exposure of password hashes through SQL Injection increases the risk of offline password guessing.

## Recommendation

Migrate password storage to Argon2id with unique salts and appropriate computational parameters.

## Remediation status

Pending. This finding will be addressed separately from SQLI-002.

## Disclosure

No recovered passwords or authentication secrets are included in this document.
