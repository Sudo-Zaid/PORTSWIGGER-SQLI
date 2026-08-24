# 🧪 Labs — all 18 PortSwigger SQL injection labs

Ordered by difficulty, exactly like the Web Security Academy. Each folder has a full write-up: objective → methodology → payload → why it works → remediation.

## 🟩 Apprentice
| # | Lab | Technique |
|---|-----|-----------|
| 01 | [WHERE clause — retrieve hidden data](lab-01-where-clause-retrieve-hidden-data/) | `'--`, `OR 1=1` |
| 02 | [Login bypass](lab-02-login-bypass/) | `administrator'--` |

## 🟨 Practitioner
| # | Lab | Technique |
|---|-----|-----------|
| 03 | [UNION — determine number of columns](lab-03-union-determine-columns/) | `ORDER BY` / `UNION SELECT NULL` |
| 04 | [UNION — find a column containing text](lab-04-union-find-text-column/) | string-in-each-column |
| 05 | [UNION — retrieve data from other tables](lab-05-union-retrieve-other-tables/) | `UNION SELECT username,password` |
| 06 | [UNION — multiple values in a single column](lab-06-union-multiple-values-single-column/) | `||` concat |
| 07 | [DB version on Oracle](lab-07-db-version-oracle/) | `v$version`, `FROM dual` |
| 08 | [DB version on MySQL / Microsoft](lab-08-db-version-mysql-mssql/) | `@@version`, `#` |
| 09 | [List DB contents (non-Oracle)](lab-09-list-db-contents-non-oracle/) | `information_schema` |
| 10 | [List DB contents (Oracle)](lab-10-list-db-contents-oracle/) | `all_tables` / `all_tab_columns` |
| 11 | [Blind — conditional responses](lab-11-blind-conditional-responses/) | boolean oracle |
| 12 | [Blind — conditional errors](lab-12-blind-conditional-errors/) | `CASE WHEN … 1/0` |
| 13 | [Visible error-based](lab-13-visible-error-based/) | `CAST(... AS int)` |
| 14 | [Blind — time delays](lab-14-blind-time-delays/) | `pg_sleep` |
| 15 | [Blind — time delays + info retrieval](lab-15-blind-time-delays-info-retrieval/) | conditional `pg_sleep` |
| 16 | [Blind — OOB interaction](lab-16-blind-oob-interaction/) | Collaborator DNS |
| 17 | [Blind — OOB data exfiltration](lab-17-blind-oob-data-exfiltration/) | data-in-subdomain |
| 18 | [Filter bypass via XML encoding](lab-18-filter-bypass-xml-encoding/) | Hackvertor `hex_entities` |

> ℹ️ Replace `LAB-ID` / `BURP-COLLAB` placeholders with your own live lab instance and Collaborator domain. Randomized table/column names differ per instance — enumerate, don't copy-paste.

⬅️ [Repo home](../README.md) · 📚 [Concepts](../concepts/README.md) · 🧰 [Cheat sheets](../cheatsheets/README.md)
