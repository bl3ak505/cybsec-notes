### 📌 A02 : Cryptographic Failures 

**Definition**

- Failures related to cryptography — or the total _absence_ of it — that expose sensitive data like passwords, credit cards, health records, or session tokens to unauthorized access.

> Previously called "Sensitive Data Exposure" — renamed because weak/missing crypto is almost always the _root cause_ of that exposure. 🔑

---

**Core Concepts**

- **Data at rest** vs **data in transit** — both need protection, both get neglected
- Weak algorithms = crypto that _looks_ secure but gets cracked fast (MD5, SHA-1, DES, RC4)
- Hardcoded keys/secrets = someone pushed their AWS key to GitHub and cried later 💀
- Improper key management = the lock is good but the key is taped to the door
- No encryption at all = plaintext passwords in DB (still happens in 2024, somehow)
- TLS misconfig = encryption exists but is misconfigured and bypassable

---

**How It Works**

- App transmits or stores sensitive data without proper encryption
- Attacker intercepts traffic (MitM) or dumps the database
    - If TLS is missing/broken → sniff credentials off the wire with Wireshark/tcpdump
    - If passwords stored as MD5 → run through hashcat/rainbow tables in minutes
    - If CBC padding oracle exists → decrypt ciphertext without the key
- Alternatively: find hardcoded API keys/secrets in source code (GitHub dorking is real 😅)
    - Search: `filename:.env DB_PASSWORD` or `AWS_SECRET_ACCESS_KEY` on GitHub
- Key derivation done wrong:
    - Password hashed with MD5 (no salt) → precomputed rainbow table cracks it instantly
    - Should be: bcrypt / Argon2 / PBKDF2 with salt + high iteration count

---

**Key Terminology**

|Term|Meaning|
|---|---|
|Encryption at rest|Data encrypted while stored on disk/DB|
|Encryption in transit|Data encrypted while moving over network (TLS/HTTPS)|
|Hashing|One-way transformation of data (passwords should be hashed, not encrypted)|
|Salting|Adding random data before hashing to prevent rainbow table attacks|
|Key derivation function (KDF)|Algorithm to derive strong keys from passwords (bcrypt, Argon2)|
|Padding Oracle|Side-channel attack that decrypts CBC-mode ciphertext without the key|
|TLS|Transport Layer Security — successor to SSL, encrypts data in transit|
|Hardcoded secret|API key/password baked directly into source code 🤦|
|Rainbow table|Precomputed hash→plaintext lookup table|
|Perfect Forward Secrecy (PFS)|Each session uses a unique key — past sessions safe if key compromised|

---

**Important Technical Details**

- **Broken algorithms to know for CEH:**
    - MD5, SHA-1 → collision-prone, crackable, DO NOT USE for passwords
    - DES (56-bit) → brute-forceable in hours
    - RC4 → biased output, broken in TLS
    - ECB mode (block cipher) → identical plaintext blocks = identical ciphertext blocks → leaks patterns (the penguin problem 🐧)
- **Strong alternatives:**
    - AES-256 (GCM mode preferred) for symmetric encryption
    - RSA-2048+ or ECC for asymmetric
    - bcrypt / Argon2id for password hashing
    - TLS 1.2 minimum, TLS 1.3 preferred
- HTTP (not HTTPS) = plaintext everything — cookies, creds, session tokens, all sniffable
- JWT with `alg: none` → literally no signature verification → forge any token you want 💀
- Sensitive data in URL params → ends up in logs, browser history, Referer headers

---

**Real-World Example**

- **RockYou breach (2009)**: 32 million passwords stored in _plaintext_. That wordlist is literally in your Kali at `/usr/share/wordlists/rockyou.txt` — a monument to cryptographic failure 😭
- **LinkedIn 2012**: 6.5M passwords hashed with unsalted SHA-1 → cracked almost entirely within days
- **JWT alg:none**: Change the algorithm field in a JWT to `"alg":"none"`, strip the signature → some servers accept it as valid — instant auth bypass

---

> 🔑 **CEH angle:** Know the difference between hashing vs encryption vs encoding (base64 is NOT encryption, it's just encoding — very testable 👀). Be able to identify weak vs strong algorithms. Understand why salting matters. TLS downgrade attacks (POODLE, BEAST, BREACH) are also fair game. 🔒