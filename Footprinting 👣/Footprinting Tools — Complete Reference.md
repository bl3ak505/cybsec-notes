# 📌 Footprinting Tools — Complete Reference (Popular + Hidden Gems)

> Every tool you need for recon, organized by passive/active and technique. CEH-aligned + real-world bug bounty ready.

---

## 🔍 Definition

**Footprinting** = Phase 1 of the hacking methodology. Systematically building a digital profile of a target before any attack. **Passive** means zero direct contact with the target. **Active** means you interact with their systems directly — traffic is generated, logs are created.

[[CEH_v13__Module_02_Footprinting_and_Reconnaissance.pdf]]

---

## 🧠 Core Concepts

- **Passive** — OSINT only, no packets sent to target, undetectable
- **Active** — direct interaction, leaves traces in firewall/IDS/server logs
- **Semi-passive** — queries third-party services (Shodan, Censys) that already scanned the target for you — technically passive but gets you "active" data
- **Attack surface** — total exposed area of a target: domains, IPs, services, employees, code
- **CT logs** — Certificate Transparency — every SSL cert issued is public and permanent, great for subdomain discovery

---

## ⚙️ How Footprinting Flows

1. **Start passive** — WHOIS, DNS, search engines, CT logs, social media — zero risk
2. **Expand passively** — subdomain enum, URL archives, GitHub leaks, cloud storage, favicon hashing
3. **Semi-passive probing** — Shodan/Censys query for target IPs, passive port data
4. **Go active** — HTTP probing, port scanning, WAF detection, web crawling, banner grabbing
5. **Synthesize** — screenshot everything alive, identify attack vectors, prioritize targets

---

## 🟢 PASSIVE — WHOIS & Domain Intel

| Tool | What It Does | Command/Usage |
|------|-------------|---------------|
| **whois** | Registrant name, email, phone, creation date, nameservers | `whois target.com` |
| **DomainTools** | Deep WHOIS history, reverse WHOIS (find all domains by same email) | Web — domaintools.com |
| **ViewDNS.info** | Reverse IP lookup, DNS history, port scan history | Web — viewdns.info |
| **SecurityTrails** | Historical DNS records, subdomain data, IP history | Web + API |
| **dnsx** | Fast multi-purpose DNS toolkit — A, MX, NS, TXT, CNAME records in bulk | `dnsx -l subs.txt -a -resp` |

---

## 🟢 PASSIVE — Search Engine & Dorking

| Tool | What It Does | Example |
|------|-------------|---------|
| **Google Dorks** | Advanced operators to surface exposed files, panels, configs | `site:target.com filetype:env` |
| **Metagoofil** | CLI Google dork tool that pulls metadata from public docs | `metagoofil -d target.com -t pdf,docx` |
| **Shodan** | Indexes internet-facing devices — finds exposed services, banners, old TLS | `shodan search "org:TargetCorp"` |
| **Censys** | Like Shodan but certificate-focused — surfaces infra through cert data | censys.io |
| **smap** | Passive port scanner — queries Shodan data, zero packets sent to target | `smap target.com` |
| **favirecon** | Hashes target favicon, queries Shodan with it — finds ALL servers with same stack | `favirecon -u https://target.com` |

> 💡 `smap` is underused af — full port data without touching the target. Perfect for staying stealth in early recon.

---

## 🟢 PASSIVE — Subdomain Enumeration

| Tool | What It Does | Command |
|------|-------------|---------|
| **Subfinder** | Fast passive subdomain enum using 40+ passive sources | `subfinder -d target.com -silent` |
| **Amass** | Deep attack surface mapping — more sources than Subfinder, does permutations too | `amass enum -passive -d target.com` |
| **Assetfinder** | Lightweight, hits crt.sh + other passive sources fast | `assetfinder --subs-only target.com` |
| **chaos-client** | Queries ProjectDiscovery's massive pre-built subdomain dataset | `chaos -d target.com -key APIKEY` |
| **crt.sh** | CT log querying directly — finds every cert ever issued for a domain | `curl "https://crt.sh/?q=%.target.com&output=json"` |
| **csprecon** | Finds subdomains via CSP headers — unique angle most tools totally miss | `csprecon -u https://target.com` |
| **AnalyticsRelationships** | Finds related domains sharing the same Google Analytics ID | `python3 analyticsrelationships.py -u https://target.com` |
| **tlsx** | TLS handshake-based subdomain discovery — finds hosts invisible to DNS | `tlsx -l subs.txt -san -cn -silent` |

---

## 🟢 PASSIVE — URL & Endpoint Archives

| Tool | What It Does | Command |
|------|-------------|---------|
| **waybackurls** | Fetches all URLs Wayback Machine ever indexed for a domain | `echo target.com \| waybackurls` |
| **gau** | Pulls URLs from AlienVault OTX, Wayback Machine, Common Crawl simultaneously | `gau target.com` |
| **hakrawler** | Crawls via archived links + sitemaps passively | `echo target.com \| hakrawler` |
| **ParamSpider** | Mines parameters from web archives — surfaces injection points without live requests | `python3 paramspider.py -d target.com` |

> 💡 `waybackurls` + `gf` is a standard bug bounty combo — dumps old params then filters for XSS/SSRF/redirect patterns before touching live app.

---

## 🟢 PASSIVE — GitHub & Code Leak Recon

| Tool | What It Does | Command |
|------|-------------|---------|
| **trufflehog** | Scans git repos + history for high-entropy strings (API keys, tokens) | `trufflehog git https://github.com/target/repo` |
| **gitleaks** | Detects secrets in git history including deleted commits | `gitleaks detect --source . -v` |
| **gitrob** | Profiles GitHub orgs and finds sensitive files committed by their devs | `gitrob analyze target_org` |
| **github-dorker** | Automates GitHub dork searches scoped to an org | `github-dorker -d target.com` |

> 💡 Manual dork: `org:TargetCorp "api_key"` or `org:TargetCorp filename:.env "password"` — still finds live creds constantly.

---

## 🟢 PASSIVE — Cloud & Infrastructure Exposure

| Tool | What It Does | Command |
|------|-------------|---------|
| **CloudEnum** | Enumerates S3 buckets, Azure blobs, GCP storage for target org name | `python3 cloud_enum.py -k target` |
| **S3Scanner** | Checks if discovered S3 buckets are public, readable, or writable | `s3scanner scan --bucket target-assets` |
| **asnmap** | Maps org's full IP range by resolving their ASN number | `asnmap -d target.com` |

---

## 🟢 PASSIVE — Email & People OSINT

| Tool | What It Does | Command |
|------|-------------|---------|
| **theHarvester** | Pulls emails, subdomains, hosts from Google, Bing, LinkedIn, PGP servers | `theHarvester -d target.com -l 200 -b google` |
| **Hunter.io** | Email pattern finder + verification | Web + API |
| **Sherlock** | Finds usernames across 300+ social platforms | `sherlock username` |
| **Infoga** | Email OSINT — domains, sources, breach data | `python3 infoga.py --domain target.com` |
| **Maltego** | Graphical OSINT + link analysis — maps relationships between people, domains, IPs | GUI — CE is free |

---

## 🟢 PASSIVE — Metadata Extraction

| Tool | What It Does | Command |
|------|-------------|---------|
| **ExifTool** | Extracts metadata from images, docs, media files | `exiftool file.jpg` |
| **FOCA** | Fingerprints orgs via metadata from collected files — usernames, paths, emails | GUI (Windows) |
| **Photon** | Crawls target URL for emails, subdomains, social links, secret keys | `python3 photon.py -u https://target.com` |

---

## 🔴 ACTIVE — Port Scanning

| Tool | What It Does | Command |
|------|-------------|---------|
| **Nmap** | The standard — port scanning, OS detection, service version, NSE scripts | `nmap -sV -O -A target.com` |
| **RustScan** | Finds open ports in seconds, pipes results to Nmap | `rustscan -a target.com -- -sV -sC` |
| **Masscan** | Internet-scale port scanning — extremely fast, full port range | `masscan -p1-65535 target.com --rate=1000` |
| **Naabu** | Go-based port scanner — fast and reliable, designed for large-scope bug bounty | `naabu -host target.com -top-ports 1000` |

---

## 🔴 ACTIVE — HTTP Probing & Web Fingerprinting

| Tool | What It Does | Command |
|------|-------------|---------|
| **httpx** | Probes web servers — status codes, titles, tech stack, response headers, TLS | `cat subs.txt \| httpx -title -status-code -tech-detect` |
| **WhatWeb** | Fingerprints CMS, frameworks, server versions from HTTP responses | `whatweb -a 3 target.com` |
| **wafw00f** | Identifies exactly which WAF is protecting the host | `wafw00f https://target.com` |
| **Wappalyzer (CLI)** | Full tech stack dump to JSON, scriptable | `wappalyzer https://target.com` |
| **Netcraft** | Tech stack, hosting history, site report — semi-passive web query | netcraft.com |

---

## 🔴 ACTIVE — DNS Enumeration

| Tool | What It Does | Command |
|------|-------------|---------|
| **dnsenum** | Automates zone transfer attempts, NS/MX/A records, subdomain brute | `dnsenum target.com` |
| **dnsrecon** | Zone transfer, DNS record enum, reverse lookup, cache snooping | `dnsrecon -d target.com -t std` |
| **fierce** | Locates non-contiguous IP space and DNS misconfigs, great pre-Nmap step | `fierce --domain target.com` |
| **massdns** | High-performance DNS resolver — brute forces subdomains at scale | `massdns -r resolvers.txt -t A -o S -w out.txt subs.txt` |
| **puredns** | DNS bruteforce + wildcard filtering using massdns under the hood | `puredns bruteforce wordlist.txt target.com` |
| **Sublist3r** | Passive subdomain enum via search engines — solid for quick early hits | `sublist3r -d target.com -o subs.txt` |
| **dig** | Manual DNS querying — zone transfers, record types, reverse lookup | `dig axfr @ns1.target.com target.com` |

---

## 🔴 ACTIVE — Web Crawling & JS Recon

| Tool | What It Does | Command |
|------|-------------|---------|
| **Katana** | Next-gen crawler — handles JS rendering, finds endpoints others miss | `katana -u https://target.com -jc -d 5` |
| **LinkFinder** | Parses JS files for hidden API endpoints and paths | `python3 linkfinder.py -i https://target.com/app.js -o cli` |
| **SecretFinder** | Hunts API keys and tokens specifically inside JS files | `python3 SecretFinder.py -i https://target.com/app.js -o cli` |
| **Arjun** | HTTP parameter discovery — finds hidden GET/POST params not visible in forms | `arjun -u https://target.com/api/endpoint` |

---

## 🔴 ACTIVE — Visual Recon

| Tool | What It Does | Command |
|------|-------------|---------|
| **Aquatone** | Visual inspection across hundreds of hosts — pipes subdomain list, outputs screenshot report | `cat subs.txt \| aquatone` |
| **EyeWitness** | Screenshots web apps, flags default credentials if detected | `eyewitness -f subs.txt --prepend-https -d screenshots/` |
| **gowitness** | Lightweight Go screenshotter with SQLite storage + built-in web UI | `gowitness file -f subs.txt` |

> 💡 200 subdomains? Don't open them manually. Pipe into Aquatone, visually triage in 10 minutes.

---

## 🟡 SEMI-PASSIVE — Subdomain Takeover Detection

| Tool | What It Does | Command |
|------|-------------|---------|
| **subjack** | Checks subdomains for dangling DNS pointing to unclaimed services | `subjack -w subs.txt -t 100 -o results.txt` |
| **dnstake** | Finds subdomains with NOERROR DNS responses pointing nowhere | `dnstake -f subs.txt` |
| **can-i-take-over-xyz** | GitHub reference list — which services are vulnerable to subdomain takeover | github.com/EdOverflow/can-i-take-over-xyz |

---

## 🟡 SEMI-PASSIVE — Automation Frameworks

| Tool | What It Does | Command |
|------|-------------|---------|
| **reconFTW** | Full pipeline — subdomain enum, DNS, WAF, port scan, JS secrets, screenshots, vuln scan | `reconftw.sh -d target.com -a` |
| **BBOT** | ProjectDiscovery's modular recon — passive + active in one clean pipeline | `bbot -t target.com -f subdomain-enum` |
| **SpiderFoot** | Automated OSINT from 200+ sources — WHOIS, subdomains, creds, dark web refs | `spiderfoot -s target.com` or web GUI |
| **Recon-ng** | Modular web recon framework — like Metasploit but for OSINT | `recon-ng` → `marketplace install all` |

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **ASN** | Autonomous System Number — identifies an org's IP block |
| **CT logs** | Certificate Transparency — public record of every SSL cert ever issued |
| **Dangling DNS** | DNS record pointing to a dead resource → subdomain takeover potential |
| **Favicon hash** | MD5/SHA of favicon — indexed by Shodan, used to fingerprint hidden infra |
| **CSP header** | Content-Security-Policy — lists approved domains, accidentally maps external services |
| **Zone transfer** | AXFR — dumps entire DNS zone; rarely works now but tested in CEH |
| **Banner grabbing** | Reading service response headers to identify software + version |
| **Semi-passive** | Querying third-party sources (Shodan/Censys) that already scanned the target |

---

## 🔥 Important Details & Gotchas

- **Zone transfers basically never work** in 2025 but CEH theory still tests them — know `dig axfr`
- **CT logs are permanent** — crt.sh finds dev/staging subdomains that were certed years ago and never properly decommissioned
- **JS files leak everything** — Katana + LinkFinder + SecretFinder on JS = API keys, internal endpoints, tokens left by devs
- **smap beats Nmap for stealth** — queries Shodan's existing data, your IP never reaches the target
- **favirecon** is slept on — targets hide Nginx versions in headers but forget their favicon reveals the same stack
- **Passive vs Active line is blurring** — Shodan/Censys do the active scanning for you, so you get port/banner data with zero direct contact
- **Always wafw00f before payloads** — sending SQLi/XSS at a WAF-protected endpoint without knowing the WAF = wasted time + potential block

---

## 🌍 Real Recon Chain (Bug Bounty / CEH Practical)

```bash
# PHASE 1 — Passive subdomain harvest
subfinder -d target.com -silent | anew subs.txt
amass enum -passive -d target.com | anew subs.txt
assetfinder --subs-only target.com | anew subs.txt
curl "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | anew subs.txt

# PHASE 2 — Probe what's alive
cat subs.txt | httpx -silent -status-code -title -tech-detect | anew live.txt

# PHASE 3 — Visual triage
cat live.txt | aquatone -out screenshots/

# PHASE 4 — WAF check
wafw00f https://target.com

# PHASE 5 — URL archive dump
cat subs.txt | gau | anew urls.txt
cat subs.txt | waybackurls | anew urls.txt

# PHASE 6 — Param hunting
cat urls.txt | gf xss | anew xss_params.txt
cat urls.txt | gf sqli | anew sqli_params.txt

# PHASE 7 — JS secret hunting
katana -list live.txt -jc -silent | grep "\.js" | sort -u | xargs -I{} python3 SecretFinder.py -i {} -o cli

# PHASE 8 — Port recon (passive first)
smap target.com
# then active if needed
rustscan -a target.com -- -sV -sC

# PHASE 9 — Subdomain takeover check
subjack -w subs.txt -t 100 -o takeover.txt
```

---

## 🔗 Related Topics

- [[DNS Enumeration]]
- [[Subdomain Takeover]]
- [[OSINT Methodology]]
- [[Bug Bounty Recon Pipeline]]
- [[Cloud Misconfigurations]]
- [[JavaScript Recon]]
- [[Banner Grabbing]]
- [[Network Scanning]]

---

## 📚 Sources / Further Reading

- ProjectDiscovery toolsuite docs — httpx, naabu, dnsx, katana, tlsx, amass
- reconFTW GitHub — full production recon pipeline reference
- OSINT Framework (osintframework.com) — categorized passive source index
- can-i-take-over-xyz — subdomain takeover fingerprint reference
