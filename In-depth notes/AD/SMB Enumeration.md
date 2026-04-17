# 📌 SMB Enumeration

> SMB is everywhere in Windows environments — and it leaks users, shares, password policies, and OS info when misconfigured. Your first pivot almost always starts here.

---

## 🔍 Definition

SMB (Server Message Block) is a network file sharing protocol running on **port 445** (and legacy 139/NetBIOS). In AD environments, SMB is used for file shares, printer access, IPC (named pipes), and inter-process communication between domain services. For attackers, SMB is the primary enumeration protocol when you first hit a Windows network — it can expose usernames, group memberships, password policies, and accessible shares even without credentials via null sessions.

---

## 🧠 Core Concepts

- **Port 445** — direct SMB over TCP. Primary target
- **Port 139** — SMB over NetBIOS (legacy). Still scanned
- **Null session** — connecting to SMB with no credentials (empty username/password). Allowed on older or misconfigured systems
- **Guest account** — sometimes enabled, allows unauthenticated read access to shares
- **Admin shares** — hidden shares: `ADMIN$` (system root), `C$` (C drive), `IPC$` (named pipes). Accessible to admins only
- **SYSVOL / NETLOGON** — special domain shares on every DC. Contains Group Policy files, scripts. Worth checking for credentials (GPP) and scripts
- **SMB Signing** — cryptographic signing of SMB packets. When disabled → NTLM relay attacks possible. Default: required on DCs, optional on workstations
- **RPC over SMB** — many AD operations (user enum, SID lookup) happen via RPC calls transported through SMB's `IPC$` named pipe
- **SAMR** — Security Account Manager Remote protocol. Used via RPC for user/group enumeration
- **Named Pipes** — IPC mechanism in SMB. Tools like rpcclient and enum4linux use these

---

## ⚙️ Enumeration Flow

```
1. Confirm SMB is open
   nmap -p 445,139 <target> --open

2. Test null session / guest access
   smbclient -N -L //<IP>
   crackmapexec smb <IP> -u '' -p ''

3. Enumerate with enum4linux-ng (full unauthenticated sweep)
   enum4linux-ng <IP> -A

4. If creds available → authenticated enum
   crackmapexec smb <IP> -u user -p pass --users --groups --shares --pass-pol

5. Browse accessible shares
   smbclient //<IP>/<share> -U user%pass

6. Check for sensitive files
   # SYSVOL for GPP credentials
   # Config files, scripts, backup files
   # Anything with passwords in cleartext
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| smbclient | Samba CLI client — connect to shares, list, upload, download |
| enum4linux / enum4linux-ng | Wrapper around SMB/RPC tools — automates user, share, group, password policy dump |
| crackmapexec (CME) / netexec (nxc) | Multi-protocol AD Swiss army knife. CME → netexec rebrand |
| smbmap | Lists shares with READ/WRITE permissions. Good at showing access rights |
| rpcclient | Direct RPC CLI — enumerate users, groups, SIDs over SMB |
| impacket smbclient.py | Impacket's SMB client. Better Kerberos support than native smbclient |
| Manspider | Spider SMB shares recursively for sensitive file content (run via Docker) |
| RID cycling | Bruteforcing RID numbers to enumerate all users including non-standard accounts |
| Password Policy | Min length, lockout threshold, complexity. Critical info before password spraying |
| SMBv1 | Legacy, insecure. Required for EternalBlue (MS17-010). Disabled by default on modern Windows |

---

## 🔥 Important Details & Gotchas

- **enum4linux is noisy and works best on older/misconfigured systems** — SMBv1 disabled? Some checks fail. Modern alternative: `enum4linux-ng` (Python rewrite)
- **Check SYSVOL on every domain engagement** — `\\<DC>\SYSVOL\<domain>\Policies\` may contain `Groups.xml` with cpassword (GPP credentials). Even with domain creds this is often readable
- **Null session may return partial data** — sometimes IPC$ allows null bind but not user enum. Try `rpcclient -U "" -N <IP>` and run `enumdomusers` manually
- **RID cycling** — even if direct user enum fails, you can bruteforce RID numbers (500 = Administrator, 501 = Guest, 1000+ = regular users) via rpcclient: `queryuser 0x$(printf '%x' $i)`
- **smbmap > smbclient for share permissions** — smbmap shows READ/WRITE status at a glance. smbclient doesn't show perms in the listing
- **SMB signing check is critical** — if signing is not required on workstations (common), NTLM relay via Responder + ntlmrelayx can give you shells without cracking anything
- **netexec > crackmapexec** — CME was rebranded to netexec (nxc). Same syntax, actively maintained
- **Guest access on ADMIN$** — if guest is enabled and you get READ on `C$` or `ADMIN$`, that's an immediate alert. Rare but worth trying
- **Password policy before spraying** — always grab the lockout threshold first. Spraying faster than the observation window resets can lock accounts domain-wide

---

## 🌍 Real-World Usage / Examples

**Initial scan and null session:**
```bash
# Scan for SMB
nmap -p 445,139 --open -sV <target_range>

# Null session share listing
smbclient -N -L //<IP>

# Null session with CME/netexec
crackmapexec smb <IP> -u '' -p ''
nxc smb <IP> -u '' -p ''

# Guest access
smbclient -L //<IP> -U guest%''
```

**Automated enum (enum4linux-ng):**
```bash
enum4linux-ng <IP> -A           # All checks
enum4linux-ng <IP> -U           # Users only
enum4linux-ng <IP> -S           # Shares only
enum4linux-ng <IP> -G           # Groups only
enum4linux-ng <IP> -P           # Password policy
```

**Authenticated enumeration (netexec):**
```bash
nxc smb <IP> -u user -p pass --users
nxc smb <IP> -u user -p pass --groups
nxc smb <IP> -u user -p pass --shares
nxc smb <IP> -u user -p pass --pass-pol

# Execute command (test admin access)
nxc smb <IP> -u user -p pass -x "whoami"

# Password spraying (check policy first)
nxc smb <IP> -u users.txt -p 'Season2025!'
```

**Browse and download from shares:**
```bash
smbclient //<IP>/<share> -U user%pass
smb: \> ls
smb: \> cd subdir
smb: \> get interesting.txt
smb: \> mget *.txt      # download multiple
smb: \> put payload.exe # upload
```

**Impacket smbclient.py (Kerberos support):**
```bash
impacket-smbclient corp.local/user:pass@<IP>
impacket-smbclient -k -no-pass corp.local/user@<IP>  # Kerberos ticket
# shares → list all shares
# use <sharename> → cd into share
```

**rpcclient (manual RPC enum):**
```bash
rpcclient -U "" -N <IP>    # null session
rpcclient -U "user%pass" <IP>

rpcclient> enumdomusers     # list all users
rpcclient> enumdomgroups    # list all groups
rpcclient> queryuser 0x1f4  # query RID 500 (Administrator)
rpcclient> querygroupmem 0x200  # get members of RID 512 (Domain Admins)
rpcclient> getdompwinfo     # password policy
```

**RID cycling (bash):**
```bash
for i in $(seq 500 1200); do
  rpcclient -N -U "" <IP> -c "queryuser 0x$(printf '%x' $i)" 2>/dev/null | grep "User Name"
done
```

**smbmap (share permissions at a glance):**
```bash
smbmap -H <IP>                          # null session
smbmap -H <IP> -u user -p pass         # authenticated
smbmap -H <IP> -u user -p pass -r      # recursive listing
```

**Check SMB signing:**
```bash
nmap --script smb-security-mode -p 445 <IP>
nxc smb <IP> --gen-relay-list relay_targets.txt
# Hosts without signing → valid relay targets
```

**SYSVOL / GPP credential check:**
```bash
# Mount SYSVOL manually
smbclient //DC/SYSVOL -U user%pass
smb: \> ls corp.local\Policies\

# Automated with CME/netexec
nxc smb <DC-IP> -u user -p pass -M gpp_password
nxc smb <DC-IP> -u user -p pass -M gpp_autologin

# Decrypt manually
gpp-decrypt <cpassword>
```

**Vulnerability scanning:**
```bash
nmap --script smb-vuln* -p 445 <IP>
nmap --script smb-vuln-ms17-010 -p 445 <IP>   # EternalBlue
```

---

## 🔗 Related Topics

- [[In-depth notes/AD/Active Directory (AD) 1|Active Directory (AD) 1]]
- [[LDAP Enumeration]]
- [[NTLM and Pass-the-Hash]]
- [[Lateral Movement]]
- [[Kerberos Authentication]]
- [[BloodHound & Attack Path Analysis]]

---

## 📚 Sources / Further Reading

- [0xdf — SMB Enumeration Cheatsheet](https://0xdf.gitlab.io/cheatsheets/smb-enum) — practical, opinionated, from HTB writeup experience
- [HackTricks — Pentesting SMB](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb) — complete command reference
- [OffensiveCyberProfessional — enum4linux](https://www.offensivecyberprofessional.com/enum4linux/) — flags and gotchas
- netexec (nxc): `https://github.com/Pennyw0rth/NetExec`
- enum4linux-ng: `https://github.com/cddmp/enum4linux-ng`
- HTB Machines: `Blue` (EternalBlue), `Legacy` (SMBv1), `Forest` (null session), `Nest` (share analysis)
