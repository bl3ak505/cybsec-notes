# 📌 Subdomain Takeover

> When a subdomain's DNS points to an external service that's been deprovisioned — attacker claims the service and owns the subdomain.

---

## 🔍 Definition

**Subdomain Takeover** happens when a DNS record (usually CNAME) points to an external platform (GitHub Pages, Heroku, AWS S3, Fastly, etc.) that no longer has content claimed for it. An attacker registers that resource and now fully controls what `sub.target.com` serves.

---

## 🧠 Core Concepts

- **Dangling DNS** — a CNAME or A record pointing to a resource that no longer exists
- **Unclaimed resource** — the external service endpoint is available for anyone to register
- **CNAME chain** — `sub.target.com → service.herokuapp.com` — if Heroku account is gone, it's takeable
- **Impact** — serve phishing pages under target's domain, steal cookies if `document.domain` is used, bypass CSP

---

## ⚙️ How It Works

1. Target has `staging.target.com CNAME targetapp.herokuapp.com`
2. Dev decommissions the Heroku app but forgets to remove the DNS record
3. `targetapp.herokuapp.com` is now available on Heroku for anyone to claim
4. Attacker creates a Heroku app with that exact name
5. `staging.target.com` now resolves to attacker-controlled content
6. Attacker can serve phishing, steal session cookies, bypass CORS

---

## 🛠️ Detection Tools

| Tool | What It Does | Command |
|------|-------------|---------|
| **subjack** | Checks subdomains for dangling CNAME → unclaimed services | `subjack -w subs.txt -t 100 -o results.txt -ssl` |
| **dnstake** | Finds NOERROR DNS responses pointing to nowhere | `dnstake -f subs.txt` |
| **nuclei** | Has takeover templates for 50+ services | `nuclei -l subs.txt -t takeovers/` |
| **can-i-take-over-xyz** | Reference list of vulnerable services + their fingerprints | github.com/EdOverflow/can-i-take-over-xyz |

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **Dangling CNAME** | CNAME record pointing to an unclaimed/deleted external resource |
| **NXDOMAIN** | Non-existent domain — clear sign of a dead DNS record |
| **NOERROR + no data** | DNS resolves but no content — possible takeover window |
| **Fingerprint** | The specific error message a service shows when a resource is unclaimed |
| **Edge case takeover** | A record pointing to a decommissioned IP now reassigned to someone else |

---

## 🔥 Important Details & Gotchas

- **Not all NXDOMAIN = takeover** — some services require specific account verification to claim
- **Fingerprints matter** — each platform has a unique "unclaimed" page (e.g. Heroku's `No such app`, GitHub Pages' `404 There isn't a GitHub Pages site here`)
- **Check can-i-take-over-xyz first** — not every service allows registration by outsiders, some are internal only
- **S3 buckets** — `NoSuchBucket` error on an S3-backed subdomain = classic takeover candidate
- **High severity in bug bounties** — most programs pay P2/P3 for this
- **Verify before reporting** — actually claim the resource in a test environment or show the fingerprint as PoC

---

## 🌍 Vulnerable Services (Common)

| Service | Fingerprint |
|---------|------------|
| GitHub Pages | `There isn't a GitHub Pages site here` |
| Heroku | `No such app` |
| AWS S3 | `NoSuchBucket` |
| Fastly | `Fastly error: unknown domain` |
| Shopify | `Sorry, this shop is currently unavailable` |
| Zendesk | `Help Center Closed` |
| Surge.sh | `project not found` |
| Netlify | `Not Found - Request ID` |

---

## 🌍 Real-World Workflow

```bash
# Step 1 — get all subdomains
subfinder -d target.com -silent | anew subs.txt
amass enum -passive -d target.com | anew subs.txt

# Step 2 — check for takeover
subjack -w subs.txt -t 100 -timeout 30 -o takeover.txt -ssl
nuclei -l subs.txt -t /root/nuclei-templates/takeovers/ -o nuclei-takeovers.txt

# Step 3 — manual verify suspicious ones
dig CNAME suspicious.target.com
curl -sk https://suspicious.target.com | grep -i "fingerprint string"
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[DNS Enumeration]]
- [[Cloud Misconfigurations]]
- [[Bug Bounty Recon Pipeline]]

---

## 📚 Sources / Further Reading

- can-i-take-over-xyz — the definitive fingerprint reference
- HackerOne disclosed reports — dozens of real subdomain takeover PoCs
- subjack GitHub — configuration and fingerprints file
