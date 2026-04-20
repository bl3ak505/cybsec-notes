# 📌 OSINT Methodology

> Structured approach to gathering publicly available intelligence on a target without direct interaction.

---

## 🔍 Definition

**OSINT (Open Source Intelligence)** is the collection and analysis of information from publicly available sources. In security, it's the backbone of passive footprinting — building a target profile using nothing but what's already out there: websites, DNS, search engines, social media, code repos, leaked databases, job postings, and more.

---

## 🧠 Core Concepts

- **Passive** — no direct contact with target, legally safe, undetectable
- **Source layering** — combine multiple sources to cross-reference and verify findings
- **Pivoting** — using one piece of info to find another (email → employee → GitHub → API key)
- **Attack surface mapping** — the end goal: a complete picture of what's exposed
- **OPSEC** — protect your own identity while conducting OSINT (use VPN, throwaway accounts, etc.)

---

## ⚙️ OSINT Methodology Steps

1. **Define scope** — what are you targeting? Domain, org, person, IP range?
2. **Passive domain recon** — WHOIS, DNS records, certificate transparency, reverse WHOIS
3. **Search engine recon** — Google dorks, Bing, cached pages, indexed docs
4. **Subdomain discovery** — Subfinder, Amass, crt.sh, chaos
5. **Employee & email OSINT** — LinkedIn, theHarvester, Hunter.io, Maltego
6. **Code & secret leaks** — GitHub dorking, trufflehog, gitleaks
7. **Cloud exposure** — S3 buckets, Azure blobs, GCP storage via CloudEnum
8. **URL & endpoint archive** — Wayback Machine, gau, waybackurls
9. **Infrastructure mapping** — Shodan, Censys, favirecon, asnmap
10. **Synthesize** — map everything into attack surface, prioritize juicy findings

---

## 🛠️ Tool Stack by Phase

| Phase | Tools |
|-------|-------|
| Domain intel | whois, SecurityTrails, DomainTools, ViewDNS |
| Subdomain enum | Subfinder, Amass, Assetfinder, crt.sh, tlsx |
| Search engine | Google Dorks, Metagoofil, Shodan, Censys |
| Email/people | theHarvester, Hunter.io, Maltego, Sherlock |
| Code leaks | trufflehog, gitleaks, gitrob, github-dorker |
| Cloud | CloudEnum, S3Scanner, favirecon |
| URL archives | waybackurls, gau, hakrawler, ParamSpider |
| Infrastructure | asnmap, smap, Shodan, Censys |
| Automation | SpiderFoot, Recon-ng, reconFTW, BBOT |

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **OSINT** | Open Source Intelligence — public data only |
| **Pivoting** | Using discovered data to find new attack surfaces |
| **Footprint** | The digital profile of a target |
| **Dorking** | Using advanced search operators to find exposed data |
| **Reverse WHOIS** | Finding all domains registered by the same email/org |
| **ASN** | Autonomous System Number — maps to an org's full IP block |
| **CT logs** | Certificate Transparency — permanent public SSL cert records |
| **Breach data** | Leaked credentials from past data breaches |

---

## 🔥 Important Details & Gotchas

- **Job postings leak tech stacks** — "We're hiring a DevOps engineer with AWS, Terraform, and Kubernetes experience" = their exact infra
- **LinkedIn is goldmine** — employee names + roles → email format guessing → credential stuffing surface
- **GitHub dorks hit constantly** — devs commit `.env`, `config.json`, `secrets.yaml` and forget
- **Metadata in documents** — PDFs and Word files often contain author names, internal paths, software versions
- **Google cache + Wayback** — pages that were taken down still exist in cache, may have exposed data
- **Reverse WHOIS** — one leaked registrant email can reveal 50 domains owned by the same org, including ones they didn't disclose in scope

---

## 🌍 OSINT Pivot Chain Example

```
target.com
  → WHOIS: admin@target.com
    → Reverse WHOIS: finds 12 other domains
      → crt.sh: finds staging.target.com, dev.target.com
        → GitHub search: org:TargetCorp finds leaked .env
          → .env has AWS_KEY + AWS_SECRET
            → CloudEnum finds open S3 buckets
              → S3 bucket has production database backup
```

---

## 🌍 Google Dork Cheatsheet

```bash
site:target.com filetype:pdf          # indexed PDFs
site:target.com inurl:admin           # admin panels
site:target.com intitle:"index of"    # open directories
site:target.com filetype:env          # .env files
site:target.com filetype:sql          # database dumps
"@target.com" filetype:xls            # employee spreadsheets
inurl:target.com intext:"password"    # exposed creds
org:TargetCorp "api_key" site:github.com  # GitHub leaks
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[DNS Enumeration]]
- [[Cloud Misconfigurations]]
- [[JavaScript Recon]]
- [[Bug Bounty Recon Pipeline]]
- [[Banner Grabbing]]

---

## 📚 Sources / Further Reading

- OSINT Framework (osintframework.com) — categorized source index
- IntelTechniques.com — Michael Bazzell's OSINT workbooks
- MITRE ATT&CK TA0043 — Reconnaissance tactic reference
