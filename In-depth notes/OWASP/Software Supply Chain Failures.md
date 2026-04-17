# A03:Software Supply Chain Failures

---

## 1. Definition

A03:2025 refers to vulnerabilities introduced through **external code, tools, or processes** that are trusted and used in building or deploying software. Instead of attacking the final app directly, the attacker targets the supply chain — injecting malicious code somewhere between the original developer and the end user.

---

## 2. Core Concepts

The software supply chain includes everything involved in building and shipping software:

- Third-party libraries and packages (npm, PyPI, Maven, etc.)
- Build and CI/CD systems (GitHub Actions, Jenkins, etc.)
- Container images and base OS layers
- Development tools and IDEs
- Code signing and update mechanisms

**The core problem:** developers inherently trust these components. That trust is what gets weaponized.

---

## 3. How It Works

**Step-by-step:**

1. Attacker identifies a trusted component in the target's supply chain
2. They compromise it via: typosquatting, hijacking a maintainer account, injecting into a legitimate repo, or poisoning build artifacts
3. Malicious code gets pulled automatically during builds by developers or CI/CD pipelines
4. Code executes with the target environment's access — secrets, cloud credentials, production systems
5. Attacker achieves: credential theft, backdoor, data exfil, or full RCE

**Attack entry points:**

- **Dependency confusion** — uploading a public package with the same name as an internal private one; package managers pull the public one by default
- **Typosquatting** — registering near-identical package names (e.g. `reqeusts` vs `requests`)
- **Maintainer account takeover** — compromising a trusted open-source maintainer's credentials
- **CI/CD injection** — modifying build scripts to exfiltrate secrets or insert backdoors during builds
- **Compromised update mechanism** — pushing malicious updates through a legitimate signed channel

---

## 4. Key Terminology

|Term|Meaning|
|---|---|
|**Supply Chain**|Full pipeline from source code to deployed app, including all tools and deps|
|**Transitive Dependency**|A dep of your dep — not imported directly, but included automatically|
|**SBOM**|Software Bill of Materials — complete inventory of components in a product|
|**Dependency Confusion**|Attack where a malicious public package name matches an internal private one|
|**Typosquatting**|Registering a package name nearly identical to a popular legit one|
|**Artifact Signing**|Cryptographically signing build outputs to verify integrity|
|**SLSA Framework**|Supply chain Levels for Software Artifacts — Google's framework to harden supply chains|

---

## 5. Key Technical Details

- Package managers auto-resolve **transitive deps** — attackers exploit this blind trust
- CI/CD runners typically hold **env vars, secrets, and cloud credentials** — high-value targets
- Build artifacts can be tampered with **between build and deployment** if not signed
- Many orgs have **no SBOM** — they can't audit what's actually running in prod

---

## 6. Real-World Example — SolarWinds (2020)

- Attackers (APT29) compromised SolarWinds' **build environment**
- Malicious DLL (SUNBURST) was injected into legitimate Orion software updates
- Updates were **cryptographically signed** by SolarWinds — appeared fully trusted
- 18,000+ orgs (including US gov agencies) installed the backdoored update
- Attackers had stealthy persistent access for **months** before detection

**Key takeaway:** the product itself wasn't vulnerable — the **build process** was. Attacker got in before the final product even existed. 💀

---

## 7. Why It Rose to A03 in 2025

- Modern apps average **500+ dependencies** — massive attack surface
- CI/CD pipelines are critical infrastructure but often under-secured
- High-profile attacks (SolarWinds, XZ Utils backdoor, npm event-stream) proved massive real-world impact
- Traditional perimeter security **doesn't catch** code entering through a trusted update or package