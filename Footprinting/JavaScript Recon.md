# 📌 JavaScript Recon

> Mining JS files for hidden endpoints, API keys, tokens, internal routes, and secrets that developers left exposed.

---

## 🔍 Definition

**JavaScript Recon** is the process of analyzing a target's JS files to extract hidden API endpoints, internal routes, hardcoded credentials, access tokens, and other secrets that developers accidentally embed in client-side code. JS files are the most overlooked but most consistently productive recon source.

---

## 🧠 Core Concepts

- **JS files are fully readable** — browsers need to execute them, so they're served publicly
- **Minified ≠ secure** — secrets in minified JS are still strings, still grep-able
- **Hidden endpoints** — internal API routes, admin paths, undocumented parameters
- **Hardcoded secrets** — API keys, tokens, passwords, AWS keys embedded by devs
- **Source maps** — `.map` files expose original unminified source including comments

---

## ⚙️ JS Recon Workflow

### Step 1 — Collect JS Files

```bash
# Crawl target and collect JS URLs
katana -u https://target.com -jc -d 5 -silent | grep "\.js$" | sort -u | anew jsfiles.txt

# From URL archives (passive)
gau target.com | grep "\.js$" | anew jsfiles.txt
waybackurls target.com | grep "\.js$" | anew jsfiles.txt

# Quick manual check
curl -s https://target.com | grep -oP 'src="[^"]+\.js[^"]*"' | cut -d'"' -f2
```

### Step 2 — Find Hidden Endpoints

```bash
# LinkFinder on each JS file
cat jsfiles.txt | xargs -I{} python3 linkfinder.py -i {} -o cli | anew js_endpoints.txt

# Or pipe directly
python3 linkfinder.py -i https://target.com/app.js -o cli
python3 linkfinder.py -d https://target.com -o cli  # crawls the whole domain
```

### Step 3 — Hunt Secrets

```bash
# SecretFinder
cat jsfiles.txt | xargs -I{} python3 SecretFinder.py -i {} -o cli | anew js_secrets.txt

# trufflehog on downloaded JS
trufflehog filesystem ./downloaded_js/

# Manual grep for common patterns
grep -r "api_key\|apikey\|api_secret\|access_token\|client_secret\|password\|Authorization" jsfiles/
grep -r "AWS_\|AZURE_\|GCP_\|GOOGLE_" jsfiles/
```

### Step 4 — Check for Source Maps

```bash
# Source maps expose original source including comments
curl -s https://target.com/app.js | grep "sourceMappingURL"
curl -s https://target.com/app.js.map  # direct check
```

---

## 🛠️ Tools

| Tool | What It Does | Command |
|------|-------------|---------|
| **Katana** | Crawls target, handles JS rendering, collects JS file URLs | `katana -u https://target.com -jc -d 5` |
| **LinkFinder** | Parses JS for API endpoints, routes, paths | `python3 linkfinder.py -i target.js -o cli` |
| **SecretFinder** | Specifically hunts API keys, tokens, credentials in JS | `python3 SecretFinder.py -i target.js -o cli` |
| **trufflehog** | High-entropy secret detection in files/repos | `trufflehog filesystem ./js/` |
| **gau/waybackurls** | Passive — finds JS file URLs from archives without crawling | `gau target.com \| grep "\.js"` |
| **subjs** | Extracts JS files from a list of subdomains | `cat subs.txt \| subjs` |

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **Source map** | `.map` file — maps minified JS back to original readable source |
| **Hardcoded secret** | Credential embedded directly in code instead of environment variables |
| **Endpoint** | API route or URL path discovered inside JS |
| **High entropy string** | Random-looking string that's likely a key/token (trufflehog detects these) |
| **Minified JS** | Code with whitespace removed for size — still grep-able |
| **Bundle** | Webpack/Rollup output — often contains all app JS in one file |

---

## 🔥 Important Details & Gotchas

- **Webpack bundles are goldmines** — one massive `bundle.js` often contains every route, API call, and sometimes dev comments
- **Source maps** — if `.map` files are served publicly, you get the original commented, readable source code
- **Check for env file exposure** — devs sometimes reference `process.env.API_KEY` but leave the `.env` file accessible too
- **Internal API routes** — JS often references `/api/v2/internal/admin` routes that aren't linked from anywhere visible
- **Third-party SDK keys** — Google Maps API keys, Stripe keys, Twilio SIDs — all commonly found in JS, all testable for scope abuse
- **Download then analyze locally** — wget or curl the JS files, analyze offline with grep/ripgrep — faster and leaves less noise

---

## 🌍 Common Secret Patterns to Grep

```bash
# API Keys
grep -E "(api[_-]?key|apikey)\s*[:=]\s*['\"][^'\"]{10,}" js/

# AWS
grep -E "AKIA[0-9A-Z]{16}" js/

# Generic tokens
grep -E "(access[_-]?token|auth[_-]?token)\s*[:=]\s*['\"][^'\"]{10,}" js/

# Passwords
grep -E "(password|passwd|pwd)\s*[:=]\s*['\"][^'\"]{4,}" js/

# Private keys
grep -E "BEGIN (RSA|EC|OPENSSH) PRIVATE KEY" js/

# JWT
grep -E "eyJ[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+" js/
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[Bug Bounty Recon Pipeline]]
- [[OSINT Methodology]]
- [[Cloud Misconfigurations]]

---

## 📚 Sources / Further Reading

- LinkFinder GitHub — GH0st3rs/LinkFinder
- SecretFinder GitHub — m4ll0k/SecretFinder
- trufflehog GitHub — trufflesecurity/trufflehog
- PortSwigger Web Security Academy — Client-side topics
