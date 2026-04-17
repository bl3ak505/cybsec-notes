### 📌 A07: Identification & Authentication Failures 

**Definition**

- Weaknesses in how an application **verifies who you are** (authentication) and **manages your session** after login — allowing attackers to impersonate legitimate users or hijack active sessions.

> basically the bouncer fell asleep and anyone can walk in wearing a fake mustache 🥸

---

**Core Concepts**

- **Authentication** = proving who you are (login)
- **Session management** = maintaining that identity after login (cookies/tokens)
- Both can fail independently — and both are in scope here
- Weak auth = attacker gets _in_ as someone else
- Broken session = attacker _stays in_ or hijacks an existing session
- Common causes: weak passwords allowed, no MFA, bad session token generation, tokens never expire

---

**How It Works**

**Credential-based attacks:**

- No account lockout → brute-force login with rockyou.txt via Burp Intruder / Hydra
- No rate limiting on `/login` → credential stuffing (leaked creds from other breaches work here)
- Weak default creds never changed → `admin:admin`, `admin:password` → instant access
- Password reset flaws → predictable token, no expiry, token in URL leaks via Referer header

**Session-based attacks:**

- Session token in URL → leaked in logs, browser history, Referer headers
- Session token never invalidated on logout → steal old cookie, still works 👀
- Token generated with weak randomness → predictable → forge a valid session
- Session fixation: attacker sets a known session ID _before_ login → victim logs in → attacker now has authenticated session with that ID
- No session timeout → a session from 3 weeks ago still works

**Broken "remember me":**

- Long-lived tokens stored insecurely → steal cookie → persistent access

---

**Key Terminology**

|Term|Meaning|
|---|---|
|Brute force|Systematically trying all possible passwords|
|Credential stuffing|Using leaked username:password combos from other breaches|
|Session hijacking|Stealing a valid session token to impersonate a user|
|Session fixation|Attacker sets a session ID before auth, reuses it after victim logs in|
|MFA|Multi-Factor Authentication — requires 2+ verification factors|
|Token entropy|Randomness/unpredictability of a session token — low entropy = guessable|
|Cookie flags|`HttpOnly` (no JS access), `Secure` (HTTPS only), `SameSite` (CSRF protection)|
|Account lockout|Temporarily disabling account after N failed login attempts|
|Password spraying|Try one common password against many accounts (avoids lockout)|

---

**Important Technical Details**

- **Hydra** for brute-forcing login forms:
    
    ```bash
    hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"
    ```
    
- **Burp Intruder** → sniper/cluster bomb mode for credential stuffing on web forms
- Session token should be: **128+ bits of cryptographically secure random data** — anything less is sus
- Missing `HttpOnly` flag → XSS can steal session cookie via `document.cookie`
- Missing `Secure` flag → session cookie sent over HTTP → sniffable
- `SameSite=Strict/Lax` → prevents CSRF from abusing session cookies
- **JWT weaknesses** (overlap with crypto failures):
    - `alg:none` → no signature = forgeable
    - Weak secret → crack with `hashcat` or `jwt_tool`
    - No expiry (`exp` claim missing) → token valid forever 💀
- Password policies matter: minimum length, no common passwords, breach-password checking (HaveIBeenPwned API)
- **Logout must invalidate server-side session** — just deleting the client-side cookie is not enough

---

**Real-World Example**

- **Slack 2015**: Session tokens exposed in URLs → ended up in server logs → attackers harvested them → account takeovers at scale
- **Credential stuffing constantly**: Every major breach (LinkedIn, Adobe, Dropbox) feeds into credential stuffing attacks on other sites — people reuse passwords religiously 😭
- **Password spray on O365**: Try `Spring2024!` against 10,000 corporate accounts — guaranteed some will hit, no lockout triggered because it's only 1 attempt per account

---

> 🔑 **CEH angle:** Know the difference between brute-force vs credential stuffing vs password spraying — all tested. Know what makes a session token secure (entropy, HttpOnly, Secure, SameSite, expiry, server-side invalidation on logout). MFA bypass techniques (OTP interception, SIM swapping) are bonus. Session fixation vs session hijacking distinction is a classic exam trap. 🔒