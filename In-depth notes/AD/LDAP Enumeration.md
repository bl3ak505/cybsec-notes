# 📌 LDAP Enumeration

> Active Directory exposes everything through LDAP — users, groups, computers, ACLs, password policies. If anonymous bind is allowed, you're enumerating before you have a single credential.

---

## 🔍 Definition

LDAP (Lightweight Directory Access Protocol) is the protocol Active Directory uses to expose its directory database. Running on **port 389** (plaintext) and **636** (LDAPS/SSL), it allows clients to query, add, modify, and delete directory objects. For pentesters, LDAP enumeration is the primary way to extract AD structure data — user lists, group memberships, SPNs for Kerberoasting targets, password policies, and more. If anonymous bind is enabled, no credentials are needed.

---

## 🧠 Core Concepts

- **Distinguished Name (DN)** — full path to an object: `CN=John Doe,OU=Sales,DC=corp,DC=local`
- **Base DN** — the starting point for LDAP queries: `DC=corp,DC=local`
- **Anonymous Bind** — connecting to LDAP without credentials. Modern AD disables this by default, but misconfigs happen
- **Null Bind** — attempting auth with empty username and password strings. Sometimes allowed
- **Object Class** — the type of an AD object: `user`, `group`, `computer`, `organizationalUnit`, etc.
- **sAMAccountName** — the Windows username (e.g., `jdoe`). The field you want for user lists
- **userAccountControl** — attribute encoding account flags: disabled, lockout, no pre-auth, etc. Critical for finding AS-REP roasting targets
- **adminCount** — attribute set to 1 on accounts that are (or were) in privileged groups. Good privesc target indicator
- **servicePrincipalName** — SPN attribute on user/computer objects. Accounts with SPNs = Kerberoastable
- **LDAPS (port 636)** — encrypted LDAP. Channel binding may block some tools (ldapdomaindump) on hardened configs
- **Global Catalog (port 3268/3269)** — partial LDAP replica for cross-domain queries in a forest

---

## ⚙️ Enumeration Flow

```
1. Identify LDAP service
   nmap -p 389,636,3268,3269 <DC-IP>

2. Test anonymous/null bind
   ldapsearch -x -H ldap://<DC-IP> -b "DC=corp,DC=local"

3. If auth required → use creds or hash
   ldapsearch -x -H ldap://<DC-IP> -D "corp\user" -w "pass" -b "DC=corp,DC=local"

4. Extract key objects
   - All users + sAMAccountName
   - SPNs for Kerberoasting
   - Accounts with DONT_REQ_PREAUTH flag (AS-REP targets)
   - Password policy
   - Domain Admins group members

5. Import into BloodHound for attack path analysis
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| ldapsearch | Command-line LDAP query tool (OpenLDAP) |
| ldapdomaindump | Dumps full AD structure to HTML/JSON — easy to read |
| windapsearch | Python script for LDAP user/group/computer enum |
| ldeep | Python LDAP tool with offline cache support |
| enum4linux-ng | Includes LDAP + SMB + RPC enum combined |
| netexec (nxc) | Successor to CrackMapExec. Has LDAP module |
| DONT_REQ_PREAUTH | UAC flag (0x400000) = pre-auth disabled → AS-REP Roasting target |
| ACCOUNTDISABLE | UAC flag (0x2) = account is disabled |
| objectSid | Security Identifier for an AD object — used to correlate domain entities |
| Base DN | DC=corp,DC=local — the root of all LDAP queries |

---

## 🔥 Important Details & Gotchas

- **Anonymous bind is rare on modern AD but still worth trying** — some orgs leave it enabled, especially on older servers
- **LDAPS + Channel Binding** — Windows Server 2019+ can enforce LDAP channel binding/signing, breaking ldapdomaindump. Use `ldeep` or `netexec` which handle this properly
- **Description fields = credential goldmine** — admins sometimes put passwords in the Description field of user objects. `nxc ldap -M get-desc-users` pulls this automatically
- **userAccountControl bitmask** — you query specific flags with the LDAP bitwise filter `(userAccountControl:1.2.840.113556.1.4.803:=<value>)`. The `803` suffix = bitwise AND
- **Password policy via LDAP** — tells you lockout threshold → informs spray frequency (don't lock accounts)
- **adminCount=1 doesn't mean currently privileged** — this attribute is set when an account is added to a protected group but not always cleared when removed. Still good to check
- **LDAP injection** — if an application queries AD using user input without sanitization, you can manipulate queries: `admin)(&(` style payloads. Less common but worth checking in web apps backed by AD

---

## 🌍 Real-World Usage / Examples

**Anonymous / null bind test:**
```bash
ldapsearch -x -H ldap://<DC-IP> -b "DC=corp,DC=local"
ldapsearch -x -H ldap://<DC-IP> -D "" -w "" -b "DC=corp,DC=local"
```

**Full dump with creds (ldapsearch):**
```bash
# All users + key attributes
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(objectClass=user)" sAMAccountName description memberOf

# All computers
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(objectClass=computer)" cn dNSHostName operatingSystem

# Domain Admins members
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(cn=Domain Admins)" member

# Kerberoastable accounts (have SPN)
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(servicePrincipalName=*))" sAMAccountName servicePrincipalName

# AS-REP Roasting targets (pre-auth disabled)
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=4194304))" sAMAccountName

# Privileged accounts (adminCount=1)
ldapsearch -x -H ldap://<DC-IP> \
  -D "corp\lowpriv" -w "Password1" \
  -b "DC=corp,DC=local" \
  "(&(objectClass=user)(adminCount=1))" sAMAccountName
```

**ldapdomaindump (generates HTML reports):**
```bash
ldapdomaindump -u 'corp\user' -p 'pass' ldap://<DC-IP> -o ./lddump/
# Opens: domain_users.html, domain_groups.html, domain_computers.html, etc.
```

**windapsearch:**
```bash
# All users
python3 windapsearch.py --dc-ip <DC-IP> -u 'corp\user' -p 'pass' -U

# Domain Admins
python3 windapsearch.py --dc-ip <DC-IP> -u 'corp\user' -p 'pass' --da

# Privileged users
python3 windapsearch.py --dc-ip <DC-IP> -u 'corp\user' -p 'pass' --privileged-users

# Unconstrained delegation computers
python3 windapsearch.py --dc-ip <DC-IP> -u 'corp\user' -p 'pass' --unconstrained-computers
```

**netexec (modern CrackMapExec):**
```bash
# Users + desc
nxc ldap <DC-IP> -u user -p pass -M get-desc-users

# BloodHound collection via LDAP
nxc ldap <DC-IP> -u user -p pass --bloodhound -c All -d corp.local --dns-server <DC-IP>

# AD CS enumeration
nxc ldap <DC-IP> -u user -p pass -M adcs

# Machine Account Quota (how many computers non-admins can join)
nxc ldap <DC-IP> -u user -p pass -M maq
```

**Nmap LDAP scripts:**
```bash
nmap -n -sV --script "ldap* and not brute" -p 389 <DC-IP>
nmap -p 389,636,3268,3269 --script ldap-rootdse <DC-IP>
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[SMB Enumeration]]
- [[BloodHound & Attack Path Analysis]]
- [[Kerberos Authentication]]
- [[AD Certificate Services (AD CS)]]
- [[NTLM and Pass-the-Hash]]

---

## 📚 Sources / Further Reading

- [HackTricks — Pentesting LDAP](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap) — command-focused reference
- [The Hacker Recipes — LDAP Recon](https://www.thehacker.recipes/ad/recon/ldap) — ldeep + netexec examples
- [Hackviser — LDAP Pentesting](https://hackviser.com/tactics/pentesting/services/ldap) — comprehensive command breakdown
- windapsearch: `https://github.com/ropnop/windapsearch`
- ldapdomaindump: `https://github.com/dirkjanm/ldapdomaindump`
- ldeep: `https://github.com/franc-pentest/ldeep`
