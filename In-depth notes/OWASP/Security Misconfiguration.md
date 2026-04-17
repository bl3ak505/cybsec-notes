### 📌 A02: Security Misconfiguration 

**Definition**

- Occurs when security settings are defined, implemented, or maintained incorrectly — or not at all — leaving systems open to attack through default configs, unnecessary features, or improper permissions.

---

**Core Concepts**

- It's the _most common_ OWASP vulnerability — literally just people being lazy/sloppy with configs 💀
- Affects every layer: web server, app server, database, cloud, network, container, framework
- Default credentials = instant game over
- Verbose error messages = free recon for attackers
- Unnecessary features/ports open = expanded attack surface

---

**How It Works**

- Dev/admin sets up a system and forgets to harden it
- Default settings ship with open access, debug modes on, or example apps still installed
- Attacker discovers these via banner grabbing, directory brute-forcing, or just... _reading the manual_
- Exploits the misconfiguration directly — no fancy CVE needed, just config abuse
    - Example: default creds (`admin:admin`) → full dashboard access
    - Example: directory listing ON → attacker browses your entire file tree
    - Example: Cloud S3 bucket set to public → your entire DB is downloadable 💀

---

**Key Terminology**

|Term|Meaning|
|---|---|
|Default credentials|Factory-set username/passwords left unchanged|
|Directory listing|Web server exposes file/folder structure|
|Verbose errors|Detailed stack traces revealing tech stack info|
|Unnecessary services|Open ports/features that shouldn't be running|
|Security hardening|Process of locking down configs to reduce attack surface|
|Principle of Least Privilege (PoLP)|Only grant minimum permissions needed|
|Cloud misconfiguration|Misconfigured AWS/GCP/Azure resources (buckets, IAM roles, etc.)|

---

**Important Technical Details**

- XML External Entity (XXE) attacks often exist because XML parsers aren't hardened
- Admin panels exposed on default paths (`/admin`, `/phpmyadmin`, `/manager/html`)
- HTTP methods not restricted — PUT/DELETE enabled on web servers unintentionally
- CORS misconfiguration → any origin can make credentialed requests to your API
- Docker/K8s with default service account tokens = lateral movement heaven
- Security headers missing (CSP, HSTS, X-Frame-Options) = bonus vulns stacked on top

---

**Real-World Example**

- **2017 Equifax Breach**: Apache Struts wasn't patched + misconfigured — exposed 147 million records 😬
- **S3 Bucket leaks**: Thousands of companies have leaked private data because S3 buckets were set to `public-read` by default during setup and never changed
- **Default Tomcat creds** (`tomcat:tomcat` on `/manager/html`) → deploy a WAR shell → RCE. Classic.

---

> 🔑 **CEH angle**: Know that A02 is detected via config reviews, vulnerability scanners (Nessus, OpenVAS), and manual inspection. Mitigations = hardening guides (CIS Benchmarks), automated config management (Ansible/Chef), and disabling anything you don't need. 🔒