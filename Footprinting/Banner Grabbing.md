# 📌 Banner Grabbing

> Reading service response headers and banners to identify software names, versions, and configurations running on a target.

---

## 🔍 Definition

**Banner Grabbing** is an active reconnaissance technique where you connect to a service and read its response header/banner — the text a service sends back when you connect to it. This reveals the software type, version, and sometimes OS information, which maps directly to known CVEs.

---

## 🧠 Core Concepts

- **Service banner** — the text/header a service sends back on connection (e.g. `SSH-2.0-OpenSSH_7.4`)
- **HTTP headers** — `Server`, `X-Powered-By`, `X-AspNet-Version` reveal stack details
- **Active technique** — you must connect to the service, generates logs on target
- **Version → CVE** — the main point: knowing the exact version lets you look up known vulns

---

## ⚙️ How It Works

1. Connect to open port
2. Service responds with banner or sends response to crafted request
3. Extract software name + version from banner
4. Cross-reference with CVE databases (NVD, Exploit-DB, searchsploit)

---

## 🛠️ Tools & Commands

### HTTP Banner Grabbing

```bash
# curl — grab HTTP response headers
curl -I https://target.com
curl -sv https://target.com 2>&1 | grep -E "Server:|X-Powered-By:|X-AspNet"

# httpx — bulk header grab across all hosts
cat live.txt | httpx -title -status-code -tech-detect -server -o headers.txt

# WhatWeb — active fingerprinting
whatweb -a 3 target.com
whatweb -a 3 -v target.com  # verbose, shows matched plugins
```

### Raw TCP Banner Grabbing

```bash
# netcat — connect and read banner
nc -v target.com 21    # FTP
nc -v target.com 22    # SSH
nc -v target.com 25    # SMTP
nc -v target.com 80    # HTTP
nc -v target.com 110   # POP3

# telnet
telnet target.com 80
# then type: GET / HTTP/1.0 [Enter][Enter]

# nmap banner scripts
nmap -sV --script=banner target.com
nmap -sV -sC -p 21,22,25,80,443 target.com
```

### Nmap Service Detection

```bash
# Basic service version detection
nmap -sV target.com

# Aggressive — OS, version, scripts, traceroute
nmap -A target.com

# Specific port range with version detection
nmap -sV -p 1-1000 target.com

# Script-based banner grab
nmap -p 80 --script http-headers target.com
nmap -p 21 --script ftp-anon,ftp-syst target.com
nmap -p 22 --script ssh-auth-methods target.com
```

### SSL/TLS Certificate Info

```bash
# openssl — grab cert info
openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -noout -text

# sslscan — full TLS analysis
sslscan target.com

# testssl.sh — comprehensive TLS check
./testssl.sh target.com
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **Banner** | Text a service returns on connection identifying itself |
| **Server header** | HTTP response header revealing web server software |
| **X-Powered-By** | HTTP header revealing backend tech (PHP, ASP.NET, etc.) |
| **Service fingerprinting** | Identifying what software is running on a port |
| **Version disclosure** | When a service reveals its exact version number |
| **CVE** | Common Vulnerabilities and Exposures — numbered vuln database |
| **NSE** | Nmap Scripting Engine — extends Nmap with service-specific scripts |

---

## 🔥 Important Details & Gotchas

- **Version in banner ≠ always patched** — admins often patch without updating the banner string
- **Modified banners** — security-conscious admins change banners to lie — `Server: Apache` with no version
- **X-Powered-By is goldmine** — often reveals exact PHP/ASP.NET version devs forgot to disable
- **Nmap `-sV` intensity** — default is `--version-intensity 7`, use lower values to be stealthier
- **SSL cert CN/SANs** — grab the cert and read Subject Alternative Names — often reveals internal hostnames, other domains, and subdomains
- **SMTP banner** — reveals exact mail server + version, useful for CVE lookup and social engineering (org name sometimes in banner)
- **FTP banner** — often contains software version and sometimes OS, may allow anonymous login

---

## 🌍 Banner to CVE Workflow

```bash
# 1. Grab the banner
nmap -sV -p 22 target.com
# Output: 22/tcp open ssh OpenSSH 7.4 (protocol 2.0)

# 2. Search for known vulns
searchsploit openssh 7.4
searchsploit -w openssh 7.4  # opens browser to Exploit-DB

# 3. Cross-reference NVD
# nvd.nist.gov → search "OpenSSH 7.4"

# 4. Check if CVE has public PoC
# github.com search: CVE-XXXX-XXXX PoC
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[Network Scanning]]
- [[DNS Enumeration]]
- [[OSINT Methodology]]

---

## 📚 Sources / Further Reading

- Nmap NSE documentation — nmap.org/nsedoc/
- searchsploit / Exploit-DB — exploitdb.com
- NVD — nvd.nist.gov
- sslscan GitHub — rbsec/sslscan
