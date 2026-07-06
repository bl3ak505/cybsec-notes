## Module 14-15

## Web Application Pentesting — Tool Cheat Sheet

Condensed from: Web Application Hacking / SQL Injection (Module 14–15)

Environment: Kali Linux | Target example: http://testfire.net/

## 1. Footprinting / Recon

| Tool Purpose Command WHOIS Domain ownership info whois.domaintools.com or similar (web) DNS Resolve DNS records online DNS lookup site Lookup (web) Netcraft Site history, tech stack, netcraft.com site report (web) hosting WhatWeb Fingerprint web whatweb <target> technologies (CMS, server, frameworks) dnsrecon DNS enumeration dnsrecon -d <target> (A/MX/NS/TXT records, subdomains) dirb Brute-force hidden dirb <target_url> directories/files via wordlist dig Manual DNS queries dig ns <target> → name servers dig SOA <target> → start of authority; also |
| --- |
| test zone transfer curl Inspect raw HTTP curl -I <target> → shows server headers/response banner, cookies, status code |

Quick recon flow: WHOIS → DNS lookup → Netcraft → WhatWeb → dnsrecon → dirb → dig → curl.


## 2. Scanning

| Tool | Purpose | Command |
| --- | --- | --- |
| Nmap (service | Open ports + | nmap -v -sS -sV <target> |
| scan) | service/version detection |   |
| Nmap | Run NSE script for HTTP | nmap -v -sS -p80 -- |
| (script/enum) | enumeration | script=http-enum <target> |

## 3. Vulnerability Analysis

| Tool Purpose Nikto Web server vuln scanner (misconfig, outdated | Command nikto -h <target> |
| --- | --- |
| software, headers) Burp Intercepting proxy + active Suite vuln scanner | Set browser proxy → enable Intercept → right-click captured request → Scan → configure scope/options → run |
| Wapiti CLI black-box web vuln scanner | wapiti -u <target> |

## 4. Brute Force / Credential Attacks

| Tool Purpose Usage |   |
| --- | --- |
| DirBuster GUI | Launch dirbuster → set target URL → browse wordlist |
| directory/file | (/usr/share/wordlists/dirbuster/...) → Start |
| brute-forcer, |   |
| recursive |   |
| search, |   |
| custom |   |
| wordlists |   |


| Tool Purpose Usage |   |
| --- | --- |
| Gobuster Fast CLI | gobuster dir -u <target> -w <wordlist_path> |
| brute-forcer |   |
| for dirs/files, |   |
| DNS |   |
| subdomains, |   |
| vhosts |   |
| (written in |   |
| Go) |   |
| Automated Burp | Capture login request → send to Intruder → mark |
| payload- Suite | payload positions → load wordlist → Start attack |
| based brute Intruder |   |
| force on |   |
| login forms |   |
| Fast multi- Hydra | hydra -l admin -P /usr/share/wordlists/rockyou.txt |
| (THC- protocol | <target_ip> http-post-form |
|   | "/login_path:username=^USER^&password=^PASS^:Invalid |
| Hydra) password | login message" |
| cracker |   |
| (HTTP, FTP, |   |
| SSH, SMB, |   |
| etc.) |   |

## Hydra command breakdown:

- l admin → fixed username

- P wordlist.txt → password list to try

- http-post-form → target protocol/form type

- "path:params:failure_string" → login URL path, POST parameters with ^USER^/^PASS^ placeholders, and text that indicates a failed login

## 5. SQL Injection Tools

## a) Manual / Burp Suite

- Burp Proxy tab — intercept and modify request parameters manually to test payloads (e.g. ' OR '1'='1).


- Burp Repeater — resend a single modified request repeatedly to test/tune injection payloads.

- Burp Intruder — automate sending many SQLi payloads across a parameter (good for blind/time-based testing).

## b) Havij (GUI automated SQLi tool)

Workflow: enter target URL → detect injection → Tables → select DB → Get Columns → select columns → Get Data.

## c) SQLMap (CLI automated SQLi tool — primary tool of this module)

Detects, exploits, and extracts data from SQLi vulnerabilities automatically; supports MySQL, MSSQL, PostgreSQL, Oracle, SQLite, WAF-bypass via tamper scripts, OS-level access in advanced cases.

| Step | Command |
| --- | --- |
| 1. Detect vulnerability / | sqlmap -u <target_url> --random-agent |
| fingerprint backend |   |
| 2. List databases | sqlmap -u <target_url> --dbs --random-agent |
| 3. List tables in a DB | sqlmap -u <target_url> -D <db_name> --tables |
| 4. List columns in a table | sqlmap -u <target_url> -D <db_name> -T |
|   | <table_name> --columns --random-agent |
| 5. Dump specific column | sqlmap -u <target_url> -D <db_name> -T |
| data | <table_name> -C <col1>,<col2> --random-agent |

## Flag reference:

- u → target URL

- -random-agent → randomizes User-Agent to dodge basic WAF/IDS detection

- -dbs → enumerate databases

- D → select database

- T → select table

- C → select column(s)


--tables / --columns → list tables / columns respectively

## 6. Client-Side Exploitation — BeEF Framework

Browser Exploitation Framework — hooks a victim's browser via injected JS (hook.js) to run recon, social-engineering, and exploit modules; integrates with Metasploit.

```
sudo apt update
SHELL
sudo apt install beef-xss # pre-installed on Kali by default
sudo beef-xss # start framework
UI: http://127.0.0.1:3000/ui/panel
```

## 7. Suggested Tool Chain (End-to-End VAPT Flow)

Recon:

whois → dig/dnsrecon → whatweb → curl

Scan:

nmap -sS -sV → nmap --script=http-enum

Enumerate:

dirb / gobuster / dirbuster

Vuln scan:

nikto → wapiti → burp suite active scan

Brute force: hydra (login forms) / burp intruder

Exploit SQLi: burp repeater (manual) → sqlmap (automated dump)

Client-side: beef-xss (XSS → browser hooking)

## 8. OWASP Top 10:2025 (Quick Reference)

- A01:2025 – Broken Access Control

- A02:2025 – Security Misconfiguration

- A03:2025 – Software Supply Chain Failures

- A04:2025 – Cryptographic Failures

- A05:2025 – Injection

- A06:2025 – Insecure Design


- A07:2025 – Authentication Failures

- A08:2025 – Software or Data Integrity Failures

- A09:2025 – Security Logging and Alerting Failures

- A10:2025 – Mishandling of Exceptional Conditions

## Key Changes from 2021

- Security Misconfiguration moved up to #2.

- Software Supply Chain Failures replaced and expanded the old "Vulnerable & Outdated Components" category.

- SSRF is no longer a separate category and is now covered under Broken Access Control.

- Mishandling of Exceptional Conditions is a new category focusing on improper error handling, fail-open conditions, and logic errors.

## 9. Core SQLi Defense (for reporting/remediation sections)

- Parameterized queries / prepared statements (primary fix)

- Safe stored procedures

- Strict input validation & allow-listing

- Least-privilege DB accounts

- WAF as a secondary control, not a replacement for fixing the code
