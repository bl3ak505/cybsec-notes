### 📌 A04: Insecure Design 

**Definition**

- Vulnerabilities that arise from **missing or flawed security controls at the design/architecture level** — not from implementation bugs, but from the system being _designed wrong_ from the start.

> This isn't "you wrote bad code" — this is "you architected a system that was never gonna be secure no matter how clean the code is" 😬

---

**Core Concepts**

- **Design flaw ≠ Implementation bug** — a design flaw can't be patched away, it requires rearchitecting
- Root cause: security wasn't considered during the _design phase_ — threat modeling skipped, requirements vague, dev just winged it
- **Threat modeling**: the practice of asking "what can go wrong?" before writing a single line of code
- **Secure design patterns**: proven architectural approaches that bake security in (defense in depth, fail securely, least privilege)
- Business logic flaws fall heavily under this — the app does _exactly_ what it was coded to do, but the logic itself is broken
- New to OWASP Top 10 in 2021 — recognizes that secure _coding_ alone isn't enough

---

**How It Works**

- Dev team skips threat modeling → ships a feature with no thought about abuse cases
    
- Attacker finds a logical path the designer never considered:
    
    - Password reset that sends OTP to email → attacker requests reset for victim → but there's **no rate limit** → brute-force the 6-digit OTP (only 1,000,000 combinations, takes minutes 😂)
    - E-commerce: "apply coupon" function has no per-user limit → stack infinite coupons → buy stuff for free
    - Multi-step checkout: skip step 2 (payment) and jump straight to step 3 (confirmation) → order placed without paying
    - "Trust the client" design: price sent in the POST request body → attacker intercepts and changes `price=999.99` to `price=0.01`
- These aren't code bugs — the feature _works as designed_. The design itself is the vulnerability.
    

---

**Key Terminology**

|Term|Meaning|
|---|---|
|Threat modeling|Structured process of identifying threats during design phase|
|Business logic flaw|Vulnerability in the _logic_ of how a feature works, not the code|
|Secure design pattern|Proven architectural approach that prevents classes of vulnerabilities|
|Defense in depth|Multiple layers of security so one failure doesn't = full compromise|
|Fail securely|System defaults to a _denied/safe_ state on error, not an open one|
|Trust boundary|Point where data crosses between trusted and untrusted zones|
|Abuse case|The evil twin of a use case — "how could this feature be abused?"|
|Rate limiting|Restricting how many times an action can be performed in a time window|
|STRIDE|Threat modeling framework: Spoofing, Tampering, Repudiation, Info Disclosure, DoS, Elevation of Privilege|

---

**Important Technical Details**

- **STRIDE model** is the go-to threat modeling framework for CEH — know it:
    - **S**poofing → auth controls
    - **T**ampering → integrity controls
    - **R**epudiation → logging/auditing
    - **I**nfo Disclosure → confidentiality controls
    - **D**enial of Service → availability controls
    - **E**levation of Privilege → access controls
- **No rate limiting** on sensitive endpoints = design flaw (OTP brute-force, login brute-force, API abuse)
- **Trusting client-side data** = design flaw (price, role, discount, quantity sent from browser and trusted by server)
- **Insecure password recovery flows** = classic insecure design (security questions, predictable tokens, no expiry)
- Skipping **input validation at the design level** (not just implementation) = letting invalid states exist in the system
- Multi-tenant apps with no proper data isolation = one tenant can access another's data → design failure
- **Secure Development Lifecycle (SDL)** / **DevSecOps** are the mitigations — security _integrated_ into every phase, not bolted on at the end

---

**Real-World Example**

- **Instagram OTP brute-force (2019)**: Password reset used a 6-digit OTP with no rate limit → researcher brute-forced it and took over any account. Pure insecure design — the OTP mechanism itself was the problem 😬
- **Airline seat selection**: Some booking systems sent the seat price in the request. Intercept with Burp, change `seatPrice=45.00` to `seatPrice=0.00` → business class for free ✈️💀
- **"Admin" role in JWT payload trusted by server**: App designed to read role from client-provided token without server-side verification → change `"role":"user"` to `"role":"admin"` → instant privilege escalation

---

> 🔑 **CEH angle:** Key distinction for exams — **insecure design** = architectural/logic flaws (can't patch, must redesign) vs **security misconfiguration** = correct design, wrong settings (can be fixed by reconfiguring). Know STRIDE. Know that threat modeling and abuse case analysis during design phase are THE mitigations. Business logic testing is a manual process — scanners won't catch it. 🔒