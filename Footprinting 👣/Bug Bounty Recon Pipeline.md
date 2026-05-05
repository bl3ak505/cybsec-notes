# 📌 Bug Bounty Recon Pipeline

> The full automated + manual recon chain used by real bug bounty hunters to map a target before hunting vulns.

---

## 🔍 Definition

A **recon pipeline** in bug bounty is a structured, often automated sequence of tools that transforms a target domain into a complete attack surface map — live subdomains, open ports, tech stack, URL endpoints, JS secrets, and param injection points — before you write a single payload.

---

## 🧠 Core Concepts

- **Breadth first** — find everything before going deep on anything
- **Anew** — de-duplication tool, use it between every pipe to avoid reprocessing
- **Chaining tools** — output of one tool = input of next (subfinder → httpx → aquatone)
- **Scope discipline** — only target what's in scope, track out-of-scope assets separately
- **Low hanging fruit** — subdomain takeovers, exposed panels, default creds come before complex vulns

---

## ⚙️ Full Pipeline

### Phase 1 — Subdomain Harvest (Passive)

```bash
subfinder -d target.com -silent | anew subs.txt
amass enum -passive -d target.com | anew subs.txt
assetfinder --subs-only target.com | anew subs.txt
curl "https://crt.sh/?q=%.target.com&output=json" | jq '.[].name_value' | sort -u | anew subs.txt
chaos -d target.com -key $CHAOS_KEY | anew subs.txt
```

### Phase 2 — Probe Live Hosts

```bash
cat subs.txt | httpx -silent -status-code -title -tech-detect -o live.txt
cat subs.txt | httpx -silent | anew live_clean.txt
```

### Phase 3 — Visual Triage

```bash
cat live_clean.txt | aquatone -out screenshots/
# OR
eyewitness -f live_clean.txt --prepend-https -d screenshots/
```

### Phase 4 — Port Scanning

```bash
# Passive first (Shodan data, zero noise)
smap target.com

# Active port scan
cat live_clean.txt | naabu -top-ports 1000 -silent | anew ports.txt
rustscan -a target.com -- -sV -sC
```

### Phase 5 — WAF & Tech Detection

```bash
wafw00f https://target.com
whatweb -a 3 target.com
cat live_clean.txt | httpx -title -tech-detect -status-code
```

### Phase 6 — URL Archive Dump

```bash
cat subs.txt | gau | anew urls.txt
cat subs.txt | waybackurls | anew urls.txt
cat live_clean.txt | hakrawler | anew urls.txt
```

### Phase 7 — Parameter Discovery

```bash
cat urls.txt | gf xss | anew xss_params.txt
cat urls.txt | gf sqli | anew sqli_params.txt
cat urls.txt | gf ssrf | anew ssrf_params.txt
cat urls.txt | gf redirect | anew redirect_params.txt
cat urls.txt | gf lfi | anew lfi_params.txt

# Active param discovery on live endpoints
arjun -u https://target.com/api/endpoint -m GET
```

### Phase 8 — JS Recon

```bash
# Crawl and collect JS files
katana -list live_clean.txt -jc -silent | grep "\.js$" | sort -u | anew jsfiles.txt

# Find endpoints in JS
cat jsfiles.txt | xargs -I{} python3 linkfinder.py -i {} -o cli | anew js_endpoints.txt

# Find secrets in JS
cat jsfiles.txt | xargs -I{} python3 SecretFinder.py -i {} -o cli | anew js_secrets.txt
```

### Phase 9 — Subdomain Takeover Check

```bash
subjack -w subs.txt -t 100 -timeout 30 -o takeover.txt -ssl
nuclei -l subs.txt -t /root/nuclei-templates/takeovers/ -o nuclei-takeovers.txt
```

### Phase 10 — Cloud Exposure

```bash
python3 cloud_enum.py -k targetcorp
s3scanner scan --bucket-file potential_buckets.txt
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **anew** | Tool that deduplicates lines — only passes new unique lines downstream |
| **gf** | Grep-based pattern matcher for URLs — has prebuilt patterns for XSS, SQLi, SSRF etc. |
| **httpx** | Fast HTTP prober — confirms which subdomains are actually alive |
| **scope** | Domains/IPs you're authorized to test |
| **out-of-scope** | Assets you must NOT test — track them to avoid accidents |
| **PoC** | Proof of Concept — demonstration that a vulnerability is exploitable |

---

## 🔥 Important Details & Gotchas

- **Passive before active** — always. Active recon against out-of-scope assets = instant ban
- **anew is critical** — without deduplication your pipeline feeds duplicate hosts into every tool, slows everything and generates noise
- **gf patterns** — install tomnomnom/gf + gf-patterns for the prebuilt XSS/SQLi/SSRF/LFI/redirect patterns
- **JS files are the best source of hidden endpoints** — internal APIs, admin routes, hardcoded tokens
- **nuclei templates** — run the full template suite after recon, many low-effort findings come from automated checks
- **Rate limiting** — some programs have aggressive WAFs that'll ban your IP if you scan too fast. Use `--rate-limit` flags

---

## 🌍 Quick One-Liner Recon Kickoff

```bash
TARGET=target.com
mkdir -p $TARGET/{subs,urls,js,screenshots}
subfinder -d $TARGET -silent | anew $TARGET/subs/subs.txt
assetfinder --subs-only $TARGET | anew $TARGET/subs/subs.txt
cat $TARGET/subs/subs.txt | httpx -silent | anew $TARGET/subs/live.txt
cat $TARGET/subs/live.txt | aquatone -out $TARGET/screenshots/
cat $TARGET/subs/live.txt | gau | anew $TARGET/urls/urls.txt
cat $TARGET/urls/urls.txt | gf xss > $TARGET/urls/xss_params.txt
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[OSINT Methodology]]
- [[Subdomain Takeover]]
- [[JavaScript Recon]]
- [[Network Scanning]]
- [[Cloud Misconfigurations]]

---

## 📚 Sources / Further Reading

- tomnomnom/gf — pattern library for URL filtering
- projectdiscovery/nuclei-templates — vuln template collection
- reconFTW GitHub — full automated pipeline reference
- nahamsec recon methodology — classic bug bounty recon guide
