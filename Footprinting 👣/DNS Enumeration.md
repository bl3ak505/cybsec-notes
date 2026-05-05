# 📌 DNS Enumeration

> Extracting DNS records to map a target's infrastructure — subdomains, mail servers, name servers, IP ranges, and misconfigs.

---

## 🔍 Definition

**DNS Enumeration** is the process of locating all DNS records for a target domain. It's a core active footprinting technique that reveals subdomains, mail infrastructure, IP mappings, and occasionally leaks internal hostnames via zone transfers.

---

## 🧠 Core Concepts

- **DNS record types** — A, AAAA, MX, NS, TXT, CNAME, PTR, SOA, SRV
- **Zone transfer (AXFR)** — full dump of all DNS records from a misconfigured name server
- **Subdomain brute force** — guessing subdomains via wordlist, resolving each to check if it exists
- **Reverse DNS (PTR)** — resolving an IP back to a hostname
- **DNS wildcard** — `*.target.com` resolves everything → must be filtered when bruteforcing

---

## ⚙️ How It Works

1. Query **A records** — maps domain to IPv4 addresses
2. Query **MX records** — identifies mail servers (often reveals third-party services)
3. Query **NS records** — reveals authoritative nameservers
4. Query **TXT records** — SPF, DKIM, DMARC, sometimes Google/AWS verification tokens
5. Attempt **AXFR zone transfer** on each NS — usually fails, but worth trying
6. **Brute force subdomains** with wordlists via massdns or puredns
7. **Passive subdomain enum** via Subfinder, Amass, crt.sh in parallel

---

## 🛠️ Tools & Commands

| Tool | Purpose | Command |
|------|---------|---------|
| **dig** | Manual DNS queries, zone transfer attempts | `dig axfr @ns1.target.com target.com` |
| **nslookup** | Basic DNS record querying | `nslookup -type=mx target.com` |
| **dnsrecon** | Full automated DNS recon | `dnsrecon -d target.com -t std` |
| **dnsenum** | Zone transfers, subdomain brute, MX/NS enum | `dnsenum target.com` |
| **fierce** | Pre-Nmap non-contiguous IP finder | `fierce --domain target.com` |
| **massdns** | High-performance DNS resolver for brute force | `massdns -r resolvers.txt -t A -o S -w out.txt subs.txt` |
| **puredns** | DNS brute with wildcard filtering | `puredns bruteforce wordlist.txt target.com` |
| **dnsx** | Bulk multi-record querying | `dnsx -l subs.txt -a -cname -resp` |
| **subfinder** | Passive subdomain discovery | `subfinder -d target.com -silent` |
| **amass** | Deep passive + active subdomain enum | `amass enum -d target.com` |

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **A record** | Maps hostname → IPv4 |
| **AAAA record** | Maps hostname → IPv6 |
| **MX record** | Mail exchange server for the domain |
| **NS record** | Authoritative nameserver for the domain |
| **TXT record** | Arbitrary text — SPF, DKIM, verification strings |
| **CNAME** | Alias pointing to another hostname |
| **PTR record** | Reverse DNS — IP → hostname |
| **SOA record** | Start of Authority — zone admin info |
| **AXFR** | Zone transfer request — dumps all DNS records |
| **Wildcard DNS** | `*.target.com` catches all subdomains — breaks brute force |

---

## 🔥 Important Details & Gotchas

- **Zone transfers are CEH exam favourites** but almost never work in real life — misconfigured old DNS servers are the exception
- **TXT records leak a ton** — SPF reveals mail providers, DKIM reveals selectors, verification tokens reveal which services the org uses (Google Workspace, Salesforce, etc.)
- **Wildcard DNS** (`*.target.com → 1.2.3.4`) will make every subdomain brute force return valid — use puredns which filters wildcards automatically
- **Passive before active** — Subfinder + crt.sh before running dnsenum/massdns
- **CNAME chains** — a subdomain pointing to another domain pointing to a cloud service = potential subdomain takeover

---

## 🌍 Real-World Usage

```bash
# Full manual DNS recon
dig any target.com                    # all records
dig axfr @ns1.target.com target.com  # zone transfer attempt
dnsrecon -d target.com -t std        # standard enum
dnsrecon -d target.com -t axfr       # zone transfer only
dnsenum --enum target.com            # full auto enum

# Passive subdomain harvest
subfinder -d target.com | anew subs.txt
curl "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u | anew subs.txt

# Brute force subdomains
puredns bruteforce /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt target.com
massdns -r resolvers.txt -t A -o S subs.txt | grep " A " | awk '{print $1}' | sed 's/\.$//'
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[Subdomain Takeover]]
- [[Network Scanning]]
- [[OSINT Methodology]]
- [[Banner Grabbing]]

---

## 📚 Sources / Further Reading

- `man dig` — built-in manual, covers all record types and AXFR syntax
- dnsrecon GitHub — full flag reference
- SecLists DNS wordlists — `/usr/share/seclists/Discovery/DNS/`
