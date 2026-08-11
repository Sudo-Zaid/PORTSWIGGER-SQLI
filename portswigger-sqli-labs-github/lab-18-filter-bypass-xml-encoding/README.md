# Lab 18 — SQL injection with filter bypass via XML encoding

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | WAF / filter bypass |
| **Concept** | [14 — Different contexts](https://portswigger.net/web-security/sql-injection) · [15 — WAF/filter bypass](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/lab-sql-injection-with-filter-bypass-via-xml-encoding |

---

## 🎯 Objective
A stock-check feature sends an **XML** body and is protected by a WAF that blocks SQL keywords. Bypass the filter with **XML character encoding**, extract the `administrator` password, and **log in**. Engine: **PostgreSQL**.

## 🧠 Background
The `storeId` in the XML body is injectable, but a WAF blocks obvious SQL (e.g. `UNION`, `SELECT`). XML parsers decode numeric character references (`&#x53;` → `S`) **before** the value reaches the query, so encoding the keywords hides them from the WAF while keeping them valid SQL.

## 🪜 Methodology
1. Find the stock check request (Burp). The body looks like:
   ```http
   POST /product/stock HTTP/2
   Content-Type: application/xml

   <?xml version="1.0" encoding="UTF-8"?>
   <stockCheck><productId>1</productId><storeId>1</storeId></stockCheck>
   ```
2. Confirm injection in `storeId`: `1 UNION SELECT NULL` gets **blocked** by the WAF ("Attack detected").
3. Install the **Hackvertor** Burp extension (BApp Store). Select the payload text, right-click → **Hackvertor → Encode → hex_entities** (or `dec_entities`) to encode the SQL as XML entities.
4. Determine columns / dump data through the encoded payload.

## 💥 Payload
Plain SQL you want to run:
```sql
1 UNION SELECT username || '~' || password FROM users
```
Wrapped with a Hackvertor tag so Burp encodes it at send time:
```xml
<storeId><@hex_entities>1 UNION SELECT username || '~' || password FROM users<@/hex_entities></storeId>
```
On the wire it becomes (illustrative):
```xml
<storeId>&#x31;&#x20;&#x55;&#x4e;&#x49;&#x4f;&#x4e;&#x20;&#x53;&#x45;&#x4c;&#x45;&#x43;&#x54; …</storeId>
```
The WAF sees only entities; PostgreSQL sees `1 UNION SELECT username || '~' || password FROM users` and returns `administrator~<password>`.

## ✅ Why it works
The WAF inspects the **raw** bytes and never sees the keyword `UNION`/`SELECT` because they're XML entities. The XML parser decodes them into real characters, so the database executes the injected query normally. Classic parser-differential bypass.

## 🛡️ Remediation
Don't rely on a keyword blocklist/WAF. **Parameterize** the query, and validate `storeId` as an integer (allowlist). See [16 — Prevention](https://portswigger.net/web-security/sql-injection) and [15 — WAF bypass](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
When two parsers disagree, encoding smuggles the payload past the filter. `#sqli` `#practitioner` `#waf-bypass` `#xml` `#hackvertor`
