# 🎯 Security Prompts — Bug Bounty & CTF

> Practical, tool-aware prompts for real exploitation. Click-to-use format. Each prompt is context-loaded — paste directly into Claude or use as your analysis framework.

---

## 🔓 IDOR / Broken Access Control

**[bug bounty | recon → exploit]**
> Tools: burp, ffuf, autorize
```
I have an API endpoint GET /api/v2/orders/{id}. The id appears to be sequential. Walk me through enumerating other users' orders — what Burp extension handles this, what rate to set, and what response differences indicate a hit vs a 403.
```

**[bug bounty | escalation]**
> Tools: burp, autorize
```
I found an IDOR — authenticated as user A, I can read user B's data at /api/profile?uid=B. How do I escalate this to a write IDOR? What endpoints and HTTP methods should I fuzz next, and what's the minimum PoC I need for a critical-severity report?
```

---

## 🌐 SSRF

**[bug bounty | recon → exploit]**
> Tools: burp collaborator, interactsh, ffuf
```
Target has a PDF export feature that takes a URL param. Blind SSRF — no direct response. Walk me through the full detection flow: collaborator setup, what internal ranges to probe, cloud metadata endpoints to try, and how to confirm impact for a report.
```

**[both | filter bypass]**
> Tools: burp, cyberchef
```
I have confirmed SSRF but 127.0.0.1 and localhost are blocked. Give me the full bypass list: alternate representations, DNS rebinding approach, URL schema tricks (file://, gopher://), and IPv6 equivalents. Which is most likely to hit the metadata endpoint on AWS?
```

---

## 💉 SQL Injection

**[both | exploitation]**
> Tools: sqlmap, burp
```
Login form, POST request, username param. WAF is blocking common payloads — single quote returns 500 but sqlmap gets blocked immediately. What manual boolean-blind technique should I try first, and how do I tamper sqlmap to bypass the WAF? Give me the exact sqlmap command.
```

**[ctf | approach]**
> Tools: sqlmap, burp
```
CTF web challenge — login form. I tried ' OR 1=1-- and got a different error than a normal wrong password. What's the triage path from here? Error-based first or straight to UNION enum? How do I find the column count without orderby triggering a WAF?
```

---

## 🔥 XSS

**[bug bounty | impact proof]**
> Tools: burp, xss hunter
```
I found reflected XSS in a search param but CSP is set to default-src 'self'. What bypass techniques apply — JSONP endpoints, script gadgets, base-uri bypass? How do I check if the target has exploitable JSONP endpoints to escalate this to account takeover?
```

**[ctf | approach]**
> Tools: burp, xss hunter
```
CTF XSS challenge, bot visits my URL after I submit. Goal is to steal admin cookie. The filter strips <script> and on* attributes. What payload bypasses this, and how do I set up a listener to catch the cookie — what's the simplest exfil method?
```

---

## 🔑 JWT Attacks

**[both | exploitation]**
> Tools: jwt_tool, burp jwt editor, hashcat
```
I have a JWT: eyJ... The header shows alg:HS256. Walk me through the full attack checklist: alg:none bypass, key confusion HS256→RS256, weak secret brute-force with hashcat (what wordlist?), and kid header injection. Which should I try first based on the target?
```

---

## 📁 LFI / Path Traversal

**[both | recon → rce]**
> Tools: ffuf, burp
```
PHP app, ?file= param, dotdot-slash payloads are returning the same page — filters are stripping traversal. What encoding tricks bypass PHP's realpath filters? If I get LFI confirmed, what's the fastest path to RCE — log poisoning, /proc/self/environ, or PHP session file inclusion?
```

---

## 🧩 SSTI

**[both | detection → rce]**
> Tools: burp, tplmap
```
Python Flask app, user input reflects in a template. {{7*7}} returned 49 — confirmed Jinja2. Give me the exact RCE payload chain: from config leak to __class__ traversal to subprocess execution. What does tplmap's auto-exploit look like for this?
```

---

## 🤖 Prompt Injection (LLM)

**[both | detection → exploit]**
> Tools: burp, manual
```
Target is an AI chatbot that processes user-submitted documents. I want to test for indirect prompt injection. Walk me through: what payload to embed in a PDF I upload, how to detect if the injection fired (what observable difference in response?), and how to escalate to agent action hijacking if the bot has email/API tools.
```

**[bug bounty | system prompt leak]**
> Tools: manual, burp
```
AI assistant app, black-box. I want to extract the system prompt. What's the priority order of techniques: direct ask, roleplay framing, multi-turn context build, encoding tricks? How do I confirm I've got the real system prompt vs a hallucinated one, and what CVSS/severity would this typically land at in a bug bounty program?
```

---

## 🔐 Broken Auth / MFA Bypass

**[bug bounty | exploitation]**
> Tools: burp, ffuf
```
App has email-based OTP login. I want to test for: response manipulation (changing MFA-required to success), OTP brute-force (is there rate limiting?), token predictability, and race condition bypass. What Burp setup do I need for the race condition test specifically — turbo intruder or repeater?
```

---

## 📄 XXE

**[both | detection → exploit]**
> Tools: burp, interactsh
```
API accepts XML input. I want to test for XXE. Give me the detection payload (internal DTD), the file read payload for /etc/passwd, and the blind OOB payload using interactsh. If the app uses a SAX parser, does that change my approach? What about SSRF via XXE?
```

---

## 🔍 Forensics (CTF)

**[ctf | triage]**
> Tools: wireshark, volatility, binwalk, exiftool
```
CTF forensics challenge — I have a .pcapng file. Walk me through triage order: what Wireshark filters to apply first, how to extract files from the capture, what creds to look for in cleartext protocols, and if I see encrypted traffic what metadata might still be useful?
```

---

## 🐧 Privesc (Linux)

**[both | post-exploit]**
> Tools: linpeas, pspy, gtfobins
```
I have a shell as www-data on a Linux box. linpeas ran but there's a lot of output. What's the priority checklist: sudo -l first? SUID binaries? Writable cron? How do I use pspy to catch processes running as root, and what's the fastest GTFOBins path if I find a SUID binary?
```

---

## ⚙️ Misconfig / Info Disclosure

**[bug bounty | recon]**
> Tools: ffuf, shodan, nuclei
```
Recon phase on a target. I want to find exposed admin panels, .git directories, .env files, and backup files. Give me the ffuf command with a good wordlist for this, a nuclei template category that covers it, and the Shodan dorks I should run against the target domain.
```

---

## 💡 How to Use These

These prompts are **context-loaded** — they include:
- Your current attack stage
- What's already confirmed or blocked  
- What tool output you have
- What the specific goal is (detection / exploitation / report-ready PoC)

That's what makes them hit harder than generic "explain SSRF" prompts. The more context you dump in, the better the output.