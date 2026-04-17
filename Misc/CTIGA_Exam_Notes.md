# 🛡️ CTIGA Certification Exam — Complete Study Notes
> All 21 topics | 4 Phases | Structured for exam success

---

# 📌 TABLE OF CONTENTS

## Phase 1 — Foundation
1. [CTI Lifecycle](#1-cti-lifecycle)
2. [MITRE ATT&CK Framework](#2-mitre-attck-framework)
3. [Diamond Model of Intrusion Analysis](#3-diamond-model-of-intrusion-analysis)
4. [Cyber Kill Chain](#4-cyber-kill-chain)
5. [STIX / TAXII](#5-stix--taxii)
6. [TLP Classifications](#6-tlp-classifications)

## Phase 2 — Core Domains
7. [Intel Governance](#7-intel-governance)
8. [CTI in SOC / Incident Response](#8-cti-in-soc--incident-response)
9. [PIR / RFI Cycle](#9-pir--rfi-cycle)
10. [Intel Maturity Model](#10-intel-maturity-model)
11. [Executive Reporting](#11-executive-reporting)
12. [Compliance Alignment](#12-compliance-alignment)

## Phase 3 — Applied Practice
13. [Reading APT Reports](#13-reading-apt-reports)
14. [MITRE ATT&CK CTI Training](#14-mitre-attck-cti-training)
15. [OpenCTI / MISP](#15-opencti--misp)
16. [SANS FOR578](#16-sans-for578)
17. [F3EAD Cycle](#17-f3ead-cycle)
18. [Writing a Mock Intel Report](#18-writing-a-mock-intel-report)

## Phase 4 — Exam Prep
19. [Practice Questions (Master Set)](#19-practice-questions-master-set)
20. [Framework Review & Comparison Table](#20-framework-review--comparison-table)
21. [Cold Attempt Strategy & Gap Analysis](#21-cold-attempt-strategy--gap-analysis)

---

# PHASE 1 — FOUNDATION

---

## 1. CTI Lifecycle

### 1️⃣ Core Concept
The **CTI (Cyber Threat Intelligence) Lifecycle** is a structured, repeating process that guides how raw data is turned into actionable intelligence. It ensures that the right information reaches the right people at the right time to support decision-making.

Think of it as: **Raw Data → Processed → Actionable Intel → Decision → Feedback → Repeat**

---

### 2️⃣ Deep Dive

The lifecycle has **6 standard phases** (some models use 5 or 7, but 6 is most common in CTI certifications):

| Phase | Name | What Happens |
|---|---|---|
| 1 | **Planning & Direction** | Define what intel is needed and why (driven by PIRs) |
| 2 | **Collection** | Gather raw data from OSINT, HUMINT, TECHINT, feeds, dark web, etc. |
| 3 | **Processing** | Clean, normalize, translate, and organize raw data into usable format |
| 4 | **Analysis** | Apply analytical frameworks (ATT&CK, Diamond, Kill Chain) to find meaning |
| 5 | **Dissemination** | Deliver intel to the right audience in the right format |
| 6 | **Feedback** | Consumers report back on usefulness; cycle restarts with updated requirements |

**Key point:** This is a **cycle**, not a one-way process. Feedback from Phase 6 directly shapes Phase 1 of the next iteration.

**Real-world application:**
- A SOC analyst notices anomalous C2 traffic → triggers collection
- Analysts gather IOCs, correlate with threat actor TTPs
- Finished report is sent to CISO and IR team
- IR team uses intel to contain breach
- Feedback: "We needed actor attribution faster" → updates PIR for next cycle

---

### 3️⃣ Key Terms & Definitions

- **CTI (Cyber Threat Intelligence):** Evidence-based knowledge about threats, including context, indicators, and recommendations
- **Raw Data:** Unprocessed information (logs, feeds, forum posts)
- **Processed Data:** Cleaned and structured data ready for analysis
- **Finished Intelligence:** Final product delivered to decision-makers
- **PIR (Priority Intelligence Requirement):** The top questions leadership needs answered
- **IOC (Indicator of Compromise):** Artifacts that indicate a breach (IP, hash, domain)
- **TTP (Tactics, Techniques, Procedures):** How threat actors operate
- **Dissemination:** The act of distributing finished intel to consumers
- **Feedback Loop:** Mechanism by which intel consumers inform analysts of gaps or quality

---

### 4️⃣ Examples

**Example 1 — Corporate CTI team:**
A bank's CTI team receives an alert about ransomware targeting the finance sector. They:
- Plan: "What ransomware groups are active? What are their TTPs?"
- Collect: Pull reports from CISA, FS-ISAC, dark web monitoring
- Process: Normalize IOCs into STIX format
- Analyze: Map to MITRE ATT&CK; identify likely actor (LockBit)
- Disseminate: Send executive brief + technical IOC feed to SOC
- Feedback: SOC says IOCs arrived too late; improve SLA

**Example 2 — Government CTI:**
National CERT receives tip about APT group targeting critical infrastructure → full lifecycle triggered → intelligence shared via TAXII to partner nations

---

### 5️⃣ Diagrams / Flow

```
┌─────────────────────────────────────────────────────┐
│                  CTI LIFECYCLE FLOW                  │
│                                                      │
│  [1. Planning] ──► [2. Collection] ──► [3. Processing]
│       ▲                                       │
│       │                                       ▼
│  [6. Feedback] ◄── [5. Dissemination] ◄── [4. Analysis]
│                                                      │
│  ↻ Cycle repeats continuously                        │
└─────────────────────────────────────────────────────┘
```

**Data types collected in Phase 2:**
- **OSINT:** Open-source (news, social media, CVE databases)
- **TECHINT:** Technical (malware samples, network logs)
- **HUMINT:** Human sources (informants, dark web contacts)
- **SIGINT:** Signals intelligence (rarely used in private sector)
- **Commercial feeds:** Recorded Future, Mandiant, CrowdStrike

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- The **6 phases in correct order** — memorize: Plan → Collect → Process → Analyze → Disseminate → Feedback
- The **role of feedback** — exams love asking which phase closes the loop
- **Which phase produces "finished intelligence"** — Answer: Dissemination delivers it; Analysis produces it
- **PIR belongs to which phase** — Answer: Planning & Direction

**Common traps:**
- ⚠️ Confusing "Processing" with "Analysis" — Processing = cleaning data; Analysis = finding meaning
- ⚠️ Thinking the lifecycle is linear — it is a **cycle** (feedback restarts it)
- ⚠️ Forgetting feedback is a phase — some candidates skip it

---

### 7️⃣ Quick Revision

- The CTI lifecycle has **6 phases**: Planning → Collection → Processing → Analysis → Dissemination → Feedback
- **Feedback** closes the loop and makes it a cycle
- **PIRs** are defined in the Planning phase
- **Finished intelligence** is produced in Analysis and delivered in Dissemination
- The lifecycle applies to **all intel types**: strategic, operational, tactical, technical

---

### 8️⃣ Practice Questions

**Q1.** Which phase of the CTI lifecycle involves converting raw logs into a structured, usable format?
- A) Collection
- B) Analysis
- **C) Processing** ✅
- D) Dissemination

**Q2.** An intelligence consumer reports that the CTI team's reports are not addressing their key security questions. Which phase should be revisited?
- **A) Planning & Direction** ✅
- B) Collection
- C) Analysis
- D) Feedback

**Q3.** Which phase directly follows Analysis in the CTI lifecycle?
- A) Collection
- B) Feedback
- **C) Dissemination** ✅
- D) Processing

**Q4.** What is the primary purpose of the Feedback phase?
- A) Deliver finished reports to stakeholders
- B) Collect additional data sources
- **C) Improve future intelligence requirements based on consumer needs** ✅
- D) Analyze threat actor TTPs

**Q5.** A CTI analyst is pulling data from OSINT sources, dark web forums, and commercial threat feeds. Which phase is this?
- A) Planning
- **B) Collection** ✅
- C) Processing
- D) Analysis

---

## 2. MITRE ATT&CK Framework

### 1️⃣ Core Concept
**MITRE ATT&CK** (Adversarial Tactics, Techniques, and Common Knowledge) is a globally accessible, curated knowledge base of **real-world adversary behaviors**. It documents how attackers operate after gaining initial access — organized into **Tactics** (the "why") and **Techniques** (the "how").

It is used by CTI analysts to describe, detect, and defend against threats using a **common language**.

---

### 2️⃣ Deep Dive

**Structure of ATT&CK:**

| Level | Definition | Example |
|---|---|---|
| **Tactic** | The adversary's goal (category) | Persistence, Lateral Movement |
| **Technique** | How they achieve the goal | T1053 – Scheduled Task |
| **Sub-technique** | More specific method | T1053.005 – Scheduled Task/Job: Cron |
| **Procedure** | Specific implementation by a real group | APT29 used PowerShell to schedule tasks |

**ATT&CK Matrices — 3 main versions:**
- **Enterprise** — Windows, Linux, macOS, Cloud, Network
- **Mobile** — Android, iOS
- **ICS** — Industrial Control Systems

**14 Tactics in ATT&CK Enterprise (in attack order):**
1. Reconnaissance
2. Resource Development
3. Initial Access
4. Execution
5. Persistence
6. Privilege Escalation
7. Defense Evasion
8. Credential Access
9. Discovery
10. Lateral Movement
11. Collection
12. Command & Control (C2)
13. Exfiltration
14. Impact

**How CTI analysts use ATT&CK:**
- **Threat profiling:** Map known threat groups to their TTPs
- **Gap analysis:** Find detection gaps in your defenses
- **Intel reporting:** Use technique IDs (e.g., T1566.001) for precision
- **Red team planning:** Simulate realistic adversary behavior
- **SOC tuning:** Write detection rules based on technique patterns

**ATT&CK Groups:** Named threat actors (e.g., APT28, Lazarus Group, FIN7) with mapped techniques

**ATT&CK Software:** Specific malware/tools used by groups (e.g., Cobalt Strike, Mimikatz)

---

### 3️⃣ Key Terms & Definitions

- **ATT&CK:** Adversarial Tactics, Techniques, and Common Knowledge
- **Tactic:** The adversary's objective at a specific attack stage
- **Technique:** The specific method used to achieve a tactic
- **Sub-technique:** A more specific variant of a technique
- **Procedure:** A real-world implementation by a specific threat actor or tool
- **TTP:** Tactics, Techniques, and Procedures — the behavioral fingerprint of a threat actor
- **Threat Group:** Named adversary tracked in ATT&CK (e.g., APT29)
- **Navigator:** MITRE's tool for visualizing ATT&CK matrices
- **Technique ID:** Unique identifier (e.g., T1059 = Command and Scripting Interpreter)

---

### 4️⃣ Examples

**Example 1 — APT28 (Fancy Bear) Profile:**
- Known for: Spearphishing (T1566.001), credential dumping (T1003), lateral movement via RDP
- CTI use: Defender maps defenses against these exact techniques

**Example 2 — Ransomware group:**
- LockBit uses: Initial Access via phishing → Execution via PowerShell → Exfiltration before encryption → Impact (T1486 – Data Encrypted for Impact)
- CTI analyst maps the campaign to ATT&CK and alerts the SOC

**Example 3 — Detection gap analysis:**
- Analyst overlays company's detection capabilities against ATT&CK
- Finds no detection for T1070 (Indicator Removal on Host)
- Recommends new SIEM rule

---

### 5️⃣ Diagrams / Flow

```
ATT&CK TECHNIQUE HIERARCHY:

Tactic: Execution
  └── Technique: T1059 – Command and Scripting Interpreter
        ├── Sub-technique: T1059.001 – PowerShell
        ├── Sub-technique: T1059.003 – Windows Command Shell
        └── Sub-technique: T1059.007 – JavaScript

ATTACK CHAIN EXAMPLE (APT campaign):
Recon → Initial Access (T1566) → Execution (T1059) → Persistence (T1053)
→ Lateral Movement (T1021) → Collection (T1005) → Exfiltration (T1041)
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- The difference between **Tactic, Technique, Sub-technique, Procedure**
- **Number of tactics** in ATT&CK Enterprise (14)
- ATT&CK is based on **real-world observations**, not theoretical threats
- How CTI analysts use ATT&CK for **threat profiling and gap analysis**
- **ATT&CK Navigator** — the visualization tool

**Common traps:**
- ⚠️ Confusing Tactics with Techniques — Tactic = goal, Technique = method
- ⚠️ Thinking ATT&CK covers the full kill chain from Recon onward — it does since 2019 (includes Recon and Resource Development)
- ⚠️ ATT&CK ≠ Kill Chain — they are different frameworks (both may be tested)

---

### 7️⃣ Quick Revision

- MITRE ATT&CK documents **real adversary behaviors** using Tactics, Techniques, Sub-techniques, and Procedures
- **14 Tactics** in Enterprise ATT&CK
- Used for threat profiling, detection gap analysis, and intel reporting
- ATT&CK Navigator is the visualization tool
- Technique IDs (e.g., T1566) are used in professional CTI reports

---

### 8️⃣ Practice Questions

**Q1.** In MITRE ATT&CK, what does a "Tactic" represent?
- A) The specific code used by malware
- **B) The adversary's goal or objective at a given phase** ✅
- C) The platform targeted by the attacker
- D) The specific tool used in an attack

**Q2.** A CTI analyst identifies that an APT group uses T1053.005. What level of ATT&CK is this?
- A) Tactic
- B) Technique
- **C) Sub-technique** ✅
- D) Procedure

**Q3.** Which ATT&CK matrix would a CTI analyst use when analyzing threats to SCADA systems?
- A) Enterprise
- B) Mobile
- **C) ICS** ✅
- D) Cloud

**Q4.** What is the primary use of MITRE ATT&CK Navigator?
- A) Searching for CVEs
- B) Running automated penetration tests
- **C) Visualizing and overlaying ATT&CK techniques on a matrix** ✅
- D) Generating STIX threat reports

**Q5.** How many tactics does the MITRE ATT&CK Enterprise matrix contain?
- A) 7
- B) 11
- **C) 14** ✅
- D) 18

---

## 3. Diamond Model of Intrusion Analysis

### 1️⃣ Core Concept
The **Diamond Model** is an analytical framework for understanding and documenting cyber intrusions. It represents every intrusion as a relationship between **4 core features**: Adversary, Capability, Infrastructure, and Victim — arranged at the corners of a diamond.

It helps analysts **pivot** between elements to uncover the full picture of an attack campaign.

---

### 2️⃣ Deep Dive

**The 4 Core Features:**

| Feature | Definition | Examples |
|---|---|---|
| **Adversary** | The threat actor behind the attack | APT28, FIN7, unknown group |
| **Capability** | Tools, malware, exploits used | Cobalt Strike, zero-day exploit |
| **Infrastructure** | Systems used to deliver/control attacks | C2 servers, phishing domains, IPs |
| **Victim** | The target of the attack | Company, individual, sector |

**Two axes:**
- **Socio-political axis:** Adversary ←→ Victim (relationship, motive)
- **Technology axis:** Capability ←→ Infrastructure (tools and delivery)

**Meta-features (additional context):**
- Timestamp, Phase (Kill Chain), Result, Direction, Methodology, Resources

**How to use the Diamond Model:**
- **Event-based analysis:** Each attack event = one diamond
- **Activity threads:** Link multiple diamonds showing how an attack evolved
- **Activity groups:** Cluster events by actor to build a profile
- **Pivoting:** If you know the infrastructure, pivot to find other victims; if you know the capability, find other actors using it

**Real-world use:**
- Analyst finds a malicious IP (infrastructure) → pivots to find what malware (capability) was delivered → identifies the actor → discovers other victims of the same campaign

---

### 3️⃣ Key Terms & Definitions

- **Adversary:** The actor who conducts the intrusion
- **Capability:** The tools, malware, or exploits leveraged
- **Infrastructure:** The physical/logical systems used (IPs, domains, servers)
- **Victim:** The target entity
- **Socio-political axis:** Connects adversary motivation with victim targeting
- **Technology axis:** Connects capability with delivery infrastructure
- **Event:** A single intrusion instance represented as one diamond
- **Activity Thread:** A sequence of events forming a campaign timeline
- **Activity Group:** A cluster of events attributed to one adversary
- **Pivoting:** Using one known element to discover others

---

### 4️⃣ Examples

**Example 1 — Ransomware event:**
- Adversary: LockBit group
- Capability: LockBit 3.0 ransomware + RDP brute force
- Infrastructure: Bulletproof hosting in Eastern Europe, C2 domain
- Victim: Regional hospital in the US

**Example 2 — Pivoting in action:**
Analyst discovers a C2 domain (infrastructure). They pivot:
- Domain → resolves to an IP → that IP hosted 3 other domains → those domains targeted 5 other companies → all victims are in the energy sector → likely same adversary

---

### 5️⃣ Diagrams / Flow

```
           ADVERSARY
              /\
             /  \
Socio-       /    \  Socio-
political   /      \  political
axis       /        \  axis
          /  DIAMOND \
VICTIM ──/────────────\── VICTIM
         \            /
Technology\          / Technology
   axis    \        /   axis
            \      /
             \    /
              \  /
               \/
          INFRASTRUCTURE
              (center)
           CAPABILITY

Axes:
◄── Socio-political ──► (Adversary ↔ Victim)
◄── Technology ──► (Capability ↔ Infrastructure)
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **The 4 core features** — Adversary, Capability, Infrastructure, Victim
- **The 2 axes** — Socio-political and Technology
- The concept of **pivoting** using the model
- How the Diamond Model differs from the Kill Chain — Diamond = WHO/WHAT/WHERE; Kill Chain = WHEN/HOW (stages)

**Common traps:**
- ⚠️ Confusing "Capability" with "Infrastructure" — Capability = the tool/malware; Infrastructure = where it's hosted/delivered from
- ⚠️ Thinking the Diamond Model shows attack stages — it does NOT; it shows relationships between elements
- ⚠️ Forgetting meta-features exist — they add context but aren't the core 4

---

### 7️⃣ Quick Revision

- Diamond Model has **4 features**: Adversary, Capability, Infrastructure, Victim
- **2 axes**: Socio-political (Adversary↔Victim) and Technology (Capability↔Infrastructure)
- Used for **pivoting** to uncover campaign details
- Each attack event = **one diamond**; multiple diamonds = activity thread/campaign
- Diamond Model = **WHO/WHAT/WHERE** — not about attack stages

---

### 8️⃣ Practice Questions

**Q1.** In the Diamond Model, which axis connects the Adversary to the Victim?
- A) Technology axis
- **B) Socio-political axis** ✅
- C) Operational axis
- D) Attribution axis

**Q2.** A CTI analyst finds a malicious domain and uses it to identify other companies targeted by the same actor. This process is called:
- A) Threat modeling
- **B) Pivoting** ✅
- C) Kill chain mapping
- D) IOC enrichment

**Q3.** Which Diamond Model feature represents the tools and malware used in an attack?
- A) Infrastructure
- B) Victim
- **C) Capability** ✅
- D) Adversary

**Q4.** A sequence of related Diamond Model events forming a timeline is called:
- A) Activity group
- **B) Activity thread** ✅
- C) Campaign cluster
- D) Threat profile

**Q5.** The Diamond Model primarily helps analysts understand:
- A) The chronological stages of an attack
- **B) The relationships between attacker, tools, infrastructure, and victim** ✅
- C) The severity scoring of vulnerabilities
- D) The legal classification of cyber incidents

---

## 4. Cyber Kill Chain

### 1️⃣ Core Concept
The **Cyber Kill Chain** is a 7-stage model developed by **Lockheed Martin** that describes the stages an attacker must complete to successfully execute a cyber attack. Breaking any link in the chain stops the attack.

Think of it as the **attacker's to-do list** — each step must succeed before moving to the next.

---

### 2️⃣ Deep Dive

**The 7 Stages:**

| Stage | Name | What the Attacker Does |
|---|---|---|
| 1 | **Reconnaissance** | Research the target (OSINT, scanning, social engineering prep) |
| 2 | **Weaponization** | Create a weapon (malware + exploit bundled together) |
| 3 | **Delivery** | Send the weapon to the target (phishing email, USB, watering hole) |
| 4 | **Exploitation** | Trigger the exploit to execute malicious code |
| 5 | **Installation** | Install a persistent backdoor or implant on the system |
| 6 | **Command & Control (C2)** | Establish communication channel back to attacker |
| 7 | **Actions on Objectives** | Accomplish the goal (steal data, deploy ransomware, destroy systems) |

**Defender's perspective — what you can do at each stage:**

| Stage | Defensive Action |
|---|---|
| Reconnaissance | Limit public footprint, honeypots |
| Weaponization | Threat intel on known malware families |
| Delivery | Email filtering, web proxies |
| Exploitation | Patch management, EDR |
| Installation | Application whitelisting, AV |
| C2 | DNS sinkholes, firewall rules, proxy inspection |
| Actions on Objectives | DLP, least privilege, backup strategy |

**Limitations of the Kill Chain:**
- Designed for **APT/external threats** — doesn't model insider threats well
- Linear model — modern attacks can skip stages or loop back
- Doesn't address **cloud, mobile, or supply chain** attacks natively
- MITRE ATT&CK was partly developed to address these gaps

---

### 3️⃣ Key Terms & Definitions

- **Kill Chain:** Military term adapted to cybersecurity — the sequence of attack stages
- **Reconnaissance:** Intelligence gathering before the attack
- **Weaponization:** Combining an exploit with a malicious payload
- **Delivery:** Method of transmitting the weapon to the victim
- **Exploitation:** Using a vulnerability to execute code
- **Installation:** Establishing persistence on the victim system
- **C2 (Command & Control):** Communication channel allowing attacker control
- **Actions on Objectives:** The ultimate goal of the attack
- **Persistence:** Mechanisms to maintain access even after reboots
- **Payload:** The malicious code that performs harmful actions

---

### 4️⃣ Examples

**Example — Phishing to Ransomware Kill Chain:**
1. **Recon:** Attacker researches company via LinkedIn, finds finance team
2. **Weaponization:** Creates Word doc with macro dropper bundling ransomware
3. **Delivery:** Sends spearphishing email to CFO's assistant
4. **Exploitation:** User opens doc, enables macros → code executes
5. **Installation:** Ransomware drops and installs; creates registry run key
6. **C2:** Malware calls back to attacker's server for encryption key
7. **Actions on Objectives:** Files encrypted; ransom note displayed

**Defender breaks the chain at Delivery:** Email gateway blocks the phishing email → attack stops at Stage 3.

---

### 5️⃣ Diagrams / Flow

```
CYBER KILL CHAIN — ATTACK FLOW

[1. Recon] ──► [2. Weaponize] ──► [3. Deliver] ──► [4. Exploit]
                                                          │
                                                          ▼
                              [7. Objectives] ◄── [6. C2] ◄── [5. Install]

DEFENDER ACTIONS:
Stage 1: Reduce attack surface (limit OSINT exposure)
Stage 3: Email/web filtering
Stage 4: Patch management, EDR
Stage 5: AV, application control
Stage 6: Firewall, DNS sinkhole
Stage 7: DLP, backups, segmentation
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- All **7 stages in correct order** — must be memorized
- **Which stage C2 occurs** — Stage 6 (after installation)
- **Defender actions** at each stage
- **Lockheed Martin** as the creator
- Differences between Kill Chain and MITRE ATT&CK

**Common traps:**
- ⚠️ Confusing "Exploitation" and "Installation" — Exploitation = code executes; Installation = persistence established
- ⚠️ Thinking C2 happens before Installation — it's the opposite (Install first, then C2)
- ⚠️ Assuming Kill Chain = ATT&CK — they are different; Kill Chain is linear stages, ATT&CK is detailed technique catalog

---

### 7️⃣ Quick Revision

- Kill Chain = **7 stages**: Recon → Weaponize → Deliver → Exploit → Install → C2 → Actions
- Created by **Lockheed Martin**
- Breaking **any one link** stops the attack
- Stage 6 (C2) allows attacker remote control of compromised system
- Kill Chain is best for **APT/external threat modeling**; ATT&CK is more granular

---

### 8️⃣ Practice Questions

**Q1.** Which stage of the Cyber Kill Chain involves the attacker bundling an exploit with a malicious payload?
- A) Reconnaissance
- **B) Weaponization** ✅
- C) Delivery
- D) Exploitation

**Q2.** An attacker installs a RAT (Remote Access Trojan) on a victim machine and establishes a connection to their server. Which two Kill Chain stages does this represent?
- A) Delivery and Exploitation
- **B) Installation and Command & Control** ✅
- C) Exploitation and Installation
- D) Reconnaissance and Delivery

**Q3.** A defender implements DNS sinkholes to redirect malicious C2 traffic. Which Kill Chain stage is being disrupted?
- A) Installation
- **B) Command & Control** ✅
- C) Actions on Objectives
- D) Delivery

**Q4.** The Cyber Kill Chain was developed by:
- A) MITRE Corporation
- **B) Lockheed Martin** ✅
- C) SANS Institute
- D) NIST

**Q5.** Which limitation is most commonly cited about the Cyber Kill Chain?
- A) It contains too many stages
- B) It does not cover network attacks
- **C) It is a linear model that doesn't account for insider threats or modern attack complexity** ✅
- D) It was designed only for government use

---

## 5. STIX / TAXII

### 1️⃣ Core Concept
**STIX** (Structured Threat Information eXpression) and **TAXII** (Trusted Automated eXchange of Intelligence Information) are complementary standards for **sharing cyber threat intelligence**:
- **STIX** = the **language/format** (what intel looks like)
- **TAXII** = the **transport protocol** (how intel is delivered)

Together they enable automated, machine-readable threat intelligence sharing between organizations.

---

### 2️⃣ Deep Dive

### STIX

**STIX 2.1** (current version) uses **JSON format** and defines **Domain Objects (SDOs):**

| STIX Object | Description |
|---|---|
| **Indicator** | Pattern that identifies suspicious activity (e.g., IP, hash) |
| **Threat Actor** | Individual or group behind an attack |
| **Attack Pattern** | TTPs (linked to ATT&CK techniques) |
| **Malware** | Malicious software description |
| **Tool** | Legitimate software used maliciously |
| **Vulnerability** | A CVE or weakness |
| **Campaign** | A series of attacks by one actor |
| **Course of Action** | Recommended defensive action |
| **Relationship** | Links between objects (e.g., "uses", "targets") |
| **Observable** | Raw data (IP, hash, URL, email) |

**STIX Bundle:** A collection of STIX objects packaged together for sharing

**STIX uses MITRE ATT&CK IDs:** Attack Pattern objects reference ATT&CK techniques

---

### TAXII

**TAXII 2.1** defines two service types:

| Service | Description |
|---|---|
| **Collection** | A set of CTI data that can be requested/queried |
| **Channel** | A publish/subscribe feed of CTI data |

**TAXII Roles:**
- **Producer:** Shares intelligence (e.g., ISAC, commercial vendor)
- **Consumer:** Receives intelligence (e.g., enterprise SOC)

**TAXII communication is over HTTPS** — REST API-based

---

**How they work together:**
1. CTI analyst creates a threat report in STIX format
2. Report is uploaded to a TAXII server
3. Subscriber organizations pull or receive the STIX bundle via TAXII
4. SIEM/TIP ingests and acts on the IOCs automatically

---

### 3️⃣ Key Terms & Definitions

- **STIX:** Structured Threat Information eXpression — JSON-based standard for CTI format
- **TAXII:** Trusted Automated eXchange of Intelligence Information — transport protocol for CTI
- **SDO (STIX Domain Object):** Core STIX objects (Indicator, Malware, Threat Actor, etc.)
- **SRO (STIX Relationship Object):** Describes relationships between SDOs
- **Observable:** Raw indicator data (IP, domain, file hash)
- **Indicator:** Observable + context ("if you see this IP, suspect this actor")
- **Bundle:** Packaged STIX objects for sharing
- **TIP (Threat Intelligence Platform):** Software that ingests and manages STIX/TAXII data (e.g., MISP, OpenCTI)
- **ISAC:** Information Sharing and Analysis Center — sector-specific intel sharing orgs

---

### 4️⃣ Examples

**Example 1 — STIX Indicator object (simplified JSON):**
```json
{
  "type": "indicator",
  "name": "Malicious IP",
  "pattern": "[ipv4-addr:value = '192.168.1.100']",
  "labels": ["malicious-activity"],
  "valid_from": "2024-01-01T00:00:00Z"
}
```

**Example 2 — ISAC sharing:**
- FS-ISAC (Financial Services ISAC) publishes a STIX bundle about banking trojan
- Member banks pull it via TAXII automatically
- Their SIEMs block the IOCs within minutes

**Example 3 — OpenCTI platform:**
- Ingests STIX/TAXII feeds
- Visualizes relationships between threat objects
- Allows analysts to enrich and re-share intelligence

---

### 5️⃣ Diagrams / Flow

```
STIX / TAXII SHARING FLOW:

[CTI Analyst] 
     │ Creates STIX Bundle (JSON)
     ▼
[TAXII Server]
     │ Hosts Collections / Channels
     ▼
[TAXII Client (Consumer)]
     │ Pulls/subscribes to STIX Bundles via HTTPS
     ▼
[SIEM / TIP / Firewall]
     │ Ingests IOCs automatically
     ▼
[Automated Blocking / Alerting]

STIX OBJECT RELATIONSHIPS:
[Threat Actor] ──uses──► [Malware]
[Malware] ──targets──► [Vulnerability]
[Campaign] ──attributed-to──► [Threat Actor]
[Indicator] ──indicates──► [Attack Pattern]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- STIX = **format/language**; TAXII = **transport/protocol** — do not mix these up
- STIX is based on **JSON** (not XML)
- TAXII uses **HTTPS** as transport
- Key STIX objects: Indicator, Threat Actor, Malware, Attack Pattern, Campaign
- **STIX 2.1 and TAXII 2.1** are the current versions

**Common traps:**
- ⚠️ STIX and TAXII are NOT the same thing — STIX = what, TAXII = how
- ⚠️ STIX is NOT XML — it's JSON (STIX 1.x used XML; STIX 2.x uses JSON)
- ⚠️ Observable ≠ Indicator — Observable is raw data; Indicator adds context/pattern

---

### 7️⃣ Quick Revision

- **STIX** = standard for structuring/formatting CTI (JSON-based)
- **TAXII** = standard for transporting/sharing CTI (HTTPS-based REST API)
- STIX objects include: Indicator, Threat Actor, Malware, Campaign, Attack Pattern, etc.
- TAXII serves data via **Collections** (query) or **Channels** (subscribe)
- Used by platforms like **MISP, OpenCTI, Anomali** for automated intel sharing

---

### 8️⃣ Practice Questions

**Q1.** What is the primary difference between STIX and TAXII?
- A) STIX is for malware analysis; TAXII is for network monitoring
- **B) STIX defines the format of threat intelligence; TAXII defines how it is shared** ✅
- C) STIX is used by governments; TAXII is for private organizations
- D) STIX is the newer standard; TAXII is deprecated

**Q2.** Which data format does STIX 2.1 use?
- A) XML
- B) CSV
- **C) JSON** ✅
- D) YAML

**Q3.** A STIX "Indicator" object differs from an "Observable" because:
- A) Indicators are used only for IP addresses
- B) Observables are JSON; Indicators are XML
- **C) Indicators add context and pattern matching; Observables are raw data** ✅
- D) Indicators are only used in government CTI sharing

**Q4.** Which transport protocol does TAXII use?
- A) FTP
- B) SMTP
- **C) HTTPS** ✅
- D) SFTP

**Q5.** An organization wants to automatically receive threat intelligence feeds from an ISAC. Which combination enables this?
- A) STIX only
- B) TAXII only
- **C) STIX for formatting + TAXII for transport** ✅
- D) OpenCTI only

---

## 6. TLP Classifications

### 1️⃣ Core Concept
**TLP (Traffic Light Protocol)** is a simple color-coded system developed by FIRST (Forum of Incident Response and Security Teams) to indicate **how sensitive information is and who is allowed to share it**. It helps intel producers control the distribution of sensitive information.

Colors represent **sharing boundaries**, not threat severity.

---

### 2️⃣ Deep Dive

**TLP 2.0 (current standard) — 4 colors:**

| Color | Who Can Receive It | Sharing Rules |
|---|---|---|
| **TLP:RED** | Named recipients only (eyes only) | Cannot be shared at all — not even within same org broadly |
| **TLP:AMBER** | Organization + need-to-know clients | Limited to the organization; clients told only on need-to-know basis |
| **TLP:AMBER+STRICT** | Organization only (no clients) | Stricter than AMBER — only within the receiving organization |
| **TLP:GREEN** | Community (specific sector/community) | Can be shared within a defined community, not publicly |
| **TLP:WHITE** (now **TLP:CLEAR**) | Unrestricted | No restrictions — can be published publicly |

> **Note:** TLP 2.0 renamed WHITE → CLEAR and added AMBER+STRICT

**TLP is applied to:**
- Threat intelligence reports
- IOC feeds
- Incident notifications
- Vulnerability disclosures
- Any sensitive information shared in the CTI community

**How TLP is used:**
- Marked in document headers, email subjects, and STIX objects
- Example: "TLP:AMBER — This report is restricted to the receiving organization"
- STIX 2.1 supports TLP as a marking definition object

**Who uses TLP:**
- CERTs and ISACs
- Government agencies (CISA, NCSC)
- Threat intelligence vendors
- CTI sharing communities (e.g., ISAOs)

---

### 3️⃣ Key Terms & Definitions

- **TLP:** Traffic Light Protocol — color-coded information sharing framework
- **FIRST:** Forum of Incident Response and Security Teams — maintains TLP standard
- **TLP:RED:** Eyes only — no resharing permitted
- **TLP:AMBER:** Org + need-to-know clients only
- **TLP:AMBER+STRICT:** Org only (no clients)
- **TLP:GREEN:** Community sharing permitted
- **TLP:CLEAR:** Unrestricted public sharing (previously TLP:WHITE)
- **Recipient:** The intended audience of the TLP-marked information
- **Marking Definition:** STIX object that applies TLP to intelligence objects

---

### 4️⃣ Examples

**Example 1:**
- A CERT shares a vulnerability report marked **TLP:RED** with a specific bank.
- The bank's CISO reads it — the bank cannot forward it to their MSSP or parent company.

**Example 2:**
- CISA publishes an advisory marked **TLP:CLEAR** (formerly WHITE).
- Any organization or individual can read and share it freely — it may appear on public websites.

**Example 3:**
- FS-ISAC shares a banking malware IOC set marked **TLP:AMBER** with member banks.
- Banks can share it internally but cannot forward to clients or partners.

**Example 4:**
- A threat intelligence vendor sends a report marked **TLP:GREEN** to their CTI community.
- Members can share it within the community (e.g., other ISACs) but NOT publicly.

---

### 5️⃣ Diagrams / Flow

```
TLP SHARING HIERARCHY (Most Restrictive → Least Restrictive):

🔴 TLP:RED
   └── Named recipients ONLY. Zero resharing.
       "This is for your eyes only."

🟠 TLP:AMBER+STRICT
   └── Your organization only. No clients.

🟡 TLP:AMBER
   └── Your org + clients on need-to-know basis only.

🟢 TLP:GREEN
   └── Your defined community (e.g., sector ISAC members).

⬜ TLP:CLEAR
   └── Public. No restrictions.
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- The **4 TLP colors** and their sharing rules (especially RED vs AMBER vs GREEN vs CLEAR)
- **TLP:RED = most restrictive** — named recipients only, no resharing
- **TLP:CLEAR** was previously called **TLP:WHITE** — know both names
- **AMBER+STRICT** is a TLP 2.0 addition — stricter than AMBER
- TLP is maintained by **FIRST**

**Common traps:**
- ⚠️ TLP:GREEN does NOT mean public — it means community only
- ⚠️ TLP:WHITE is OLD terminology — current standard uses TLP:CLEAR
- ⚠️ TLP is about SHARING PERMISSION, not threat severity/priority
- ⚠️ TLP:AMBER allows sharing with clients on need-to-know — AMBER+STRICT does NOT allow even that

---

### 7️⃣ Quick Revision

- TLP = **color-coded system** for controlling CTI sharing; maintained by **FIRST**
- **RED** = named recipients only (most restrictive)
- **AMBER** = org + need-to-know clients; **AMBER+STRICT** = org only
- **GREEN** = community sharing; **CLEAR** = unrestricted public
- TLP 2.0 renamed WHITE → CLEAR and added AMBER+STRICT

---

### 8️⃣ Practice Questions

**Q1.** A CTI analyst receives a report marked TLP:RED. What should they do?
- A) Share it with all SOC analysts in the organization
- B) Share it with trusted ISAC partners
- **C) Keep it restricted to the named recipients only — no resharing** ✅
- D) Publish it internally on the company wiki

**Q2.** Which TLP level allows sharing within a specific sector community but not publicly?
- A) TLP:RED
- B) TLP:AMBER
- **C) TLP:GREEN** ✅
- D) TLP:CLEAR

**Q3.** What is TLP:WHITE known as in TLP version 2.0?
- A) TLP:PUBLIC
- B) TLP:OPEN
- **C) TLP:CLEAR** ✅
- D) TLP:FREE

**Q4.** What is the difference between TLP:AMBER and TLP:AMBER+STRICT?
- A) AMBER+STRICT allows sector sharing; AMBER does not
- **B) AMBER allows sharing with clients on need-to-know; AMBER+STRICT restricts to the organization only** ✅
- C) AMBER+STRICT is the older version
- D) There is no practical difference

**Q5.** TLP is maintained by which organization?
- A) MITRE
- B) NIST
- **C) FIRST** ✅
- D) CISA

---

# PHASE 2 — CORE DOMAINS

---

## 7. Intel Governance

### 1️⃣ Core Concept
**Intel Governance** refers to the **policies, processes, ethical frameworks, and legal boundaries** that guide how a CTI program operates. It ensures that threat intelligence activities are lawful, ethical, consistent, and aligned with organizational objectives.

Without governance, a CTI program may collect data illegally, share intel inappropriately, or create liability for the organization.

---

### 2️⃣ Deep Dive

**Key components of CTI governance:**

**1. Policy Framework**
- Defines what data can be collected, how, and by whom
- Specifies acceptable sources (OSINT, commercial, dark web)
- Sets data retention and destruction rules
- Establishes data classification levels

**2. Legal & Regulatory Compliance**
- **GDPR/DPDP:** Collecting data on EU/Indian citizens requires lawful basis
- **CFAA (US):** Restricts unauthorized access — even for intelligence purposes
- **Computer Misuse Act (UK):** Similar to CFAA
- Active defense (hacking back) is illegal in most jurisdictions
- Dark web research must not involve purchasing illegal goods

**3. Ethics in CTI**
- Avoid doxxing (exposing private individual info)
- Handle attribution carefully — wrong attribution causes diplomatic and legal issues
- Respect TLP markings from sharing partners
- Do not harm innocent third parties while gathering intelligence

**4. Intelligence Sharing Governance**
- Define what can be shared externally (TLP, NDAs)
- Establish vetting processes for sharing partners
- Maintain audit trails of what was shared and with whom

**5. Roles and Responsibilities**
- **CTI Manager:** Oversees program alignment with business objectives
- **CTI Analyst:** Produces and analyzes intelligence
- **Legal Counsel:** Advises on regulatory constraints
- **CISO:** Approves governance policies

---

### 3️⃣ Key Terms & Definitions

- **Governance:** The framework of rules, policies, and processes that guide CTI activities
- **Data Classification:** Categorizing data by sensitivity (e.g., Public, Internal, Confidential, Restricted)
- **Attribution:** Identifying the threat actor responsible for an attack
- **Active Defense:** Countermeasures that may cross into offensive action — legally grey/risky
- **Hack-back:** Offensive retaliation against an attacker — **illegal in most countries**
- **GDPR:** General Data Protection Regulation — EU law governing personal data handling
- **CFAA:** Computer Fraud and Abuse Act — US law against unauthorized computer access
- **NDA:** Non-Disclosure Agreement — legal contract governing information sharing
- **PIR:** Priority Intelligence Requirement — governed by what leadership approves

---

### 4️⃣ Examples

**Example 1 — Legal issue:**
A CTI analyst wants to monitor a hacker's Telegram channel. If the channel is public, it's OSINT. If the analyst creates a fake persona to join a private group, this may violate platform ToS and potentially the CFAA.

**Example 2 — Attribution ethics:**
A CTI team publicly attributes an attack to a specific nation-state based on weak evidence. If wrong, this causes a diplomatic incident. Governance requires attribution to be reviewed by senior leadership and legal before publication.

**Example 3 — Data sharing governance:**
A company wants to share malware IOCs with a partner. Governance dictates: check TLP level, confirm partner is vetted, use STIX/TAXII, document what was shared and when.

---

### 5️⃣ Diagrams / Flow

```
CTI GOVERNANCE FRAMEWORK:

          ┌─────────────────────────────┐
          │      ORGANIZATIONAL         │
          │      OBJECTIVES             │
          └──────────────┬──────────────┘
                         │
          ┌──────────────▼──────────────┐
          │     GOVERNANCE POLICIES     │
          │  (Legal, Ethics, Data Class)│
          └──────────────┬──────────────┘
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
 [Collection       [Analysis &      [Sharing &
  Boundaries]       Attribution]     Dissemination]
 Legal sources,    Ethics review,   TLP, NDAs,
 OSINT limits      attribution SLA  audit trails
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **Hack-back is illegal** in most jurisdictions
- **Attribution must be handled carefully** — legal and ethical implications
- **GDPR/CFAA** as relevant laws constraining CTI collection
- TLP as a governance tool for sharing
- The role of **legal counsel** in CTI governance

**Common traps:**
- ⚠️ Active defense ≠ Hack-back — Active defense can be legal (honeypots); hack-back generally is not
- ⚠️ Dark web research is NOT illegal by itself — accessing illegal content or transactions is
- ⚠️ Attribution without sufficient evidence is an ethics violation

---

### 7️⃣ Quick Revision

- Intel governance = **policies, ethics, legal compliance** governing CTI operations
- **Hack-back is illegal** in most jurisdictions
- Key laws: **GDPR, CFAA, Computer Misuse Act**
- Attribution requires **high confidence + legal review** before publication
- TLP and NDAs are governance tools for controlling intel sharing

---

### 8️⃣ Practice Questions

**Q1.** A CTI analyst wants to retaliate against an attacker by compromising their infrastructure. This is called:
- A) Active defense
- **B) Hack-back** ✅
- C) Threat hunting
- D) Deception operations

**Q2.** Which law primarily governs unauthorized computer access in the United States?
- **A) CFAA** ✅
- B) GDPR
- C) HIPAA
- D) SOX

**Q3.** A CTI team publicly attributes an attack to a nation-state with low-confidence evidence. This is a violation of:
- A) STIX standards
- B) TLP protocol
- **C) CTI ethics and attribution best practices** ✅
- D) Kill Chain methodology

**Q4.** Which of the following is a legally acceptable CTI collection activity?
- A) Hacking into a threat actor's infrastructure to extract IOCs
- B) Purchasing malware samples from the dark web
- **C) Monitoring public hacker forums and Telegram channels** ✅
- D) Accessing a competitor's internal systems for threat data

**Q5.** What is the primary role of governance in a CTI program?
- **A) Ensure CTI activities are lawful, ethical, and aligned with organizational objectives** ✅
- B) Maximize the number of IOCs collected
- C) Automate threat intelligence sharing
- D) Replace the need for a CISO

---

## 8. CTI in SOC / Incident Response

### 1️⃣ Core Concept
**CTI integration with the SOC and Incident Response (IR)** means embedding threat intelligence into security operations to make detections faster, investigations more precise, and responses more effective. CTI transforms the SOC from reactive (respond to alerts) to proactive (hunt for threats before they trigger alerts).

---

### 2️⃣ Deep Dive

**How CTI supports the SOC:**

| SOC Activity | CTI Contribution |
|---|---|
| Alert triage | IOCs help determine if an alert is a true positive |
| Threat hunting | TTPs from CTI guide where to look for hidden threats |
| Detection engineering | CTI-informed SIEM rules based on adversary behavior |
| Vulnerability prioritization | CTI shows which CVEs are actively exploited |
| Incident investigation | CTI provides actor attribution and behavioral context |

**How CTI supports Incident Response:**

| IR Phase | CTI Role |
|---|---|
| **Preparation** | Pre-position IOCs, build playbooks for known actors |
| **Detection & Analysis** | Enrich alerts with CTI; identify actor, tools, likely next steps |
| **Containment** | Block CTI-identified C2 domains, IPs; isolate affected systems |
| **Eradication** | Remove actor-specific persistence mechanisms identified by CTI |
| **Recovery** | Understand if actor has backup access (based on known TTPs) |
| **Lessons Learned** | Feed new IOCs/TTPs back into CTI program |

**Types of CTI consumed by SOC/IR:**

| Type | Description | SOC Use |
|---|---|---|
| **Strategic** | High-level trends, actor motivations | CISO, risk decisions |
| **Operational** | Campaign details, actor TTPs | IR team playbooks |
| **Tactical** | IOCs (IPs, hashes, domains) | SIEM rules, blocking |
| **Technical** | Malware samples, CVE details | Malware analyst, EDR tuning |

**CTI-SOC integration tools:**
- **SIEM** (Splunk, QRadar) — ingests IOC feeds
- **SOAR** (Cortex XSOAR, Splunk SOAR) — automates CTI-driven responses
- **TIP** (OpenCTI, MISP, Anomali) — manages and enriches CTI
- **EDR** (CrowdStrike, SentinelOne) — applies behavioral CTI for detection

---

### 3️⃣ Key Terms & Definitions

- **SOC:** Security Operations Center — team monitoring and responding to threats
- **IR:** Incident Response — structured process for handling security incidents
- **IOC Enrichment:** Adding context to raw indicators (e.g., IP → which actor, which campaign)
- **Threat Hunting:** Proactively searching for threats using hypothesis driven by CTI
- **SOAR:** Security Orchestration, Automation, and Response
- **TIP:** Threat Intelligence Platform — manages, normalizes, and distributes CTI
- **Tactical CTI:** Specific, short-lived indicators (IOCs)
- **Operational CTI:** Campaign-level intelligence (actor behavior, TTPs)
- **Strategic CTI:** Long-term threat trends for leadership decisions
- **Playbook:** Documented IR procedure for a specific threat scenario

---

### 4️⃣ Examples

**Example 1 — IOC-driven alert triage:**
SIEM fires an alert on a suspicious IP. CTI analyst checks the TIP — the IP is linked to LockBit infrastructure. This elevates the alert to Priority 1 immediately.

**Example 2 — Threat hunting with CTI:**
CTI reports that APT29 is using T1059.001 (PowerShell) with specific encoded commands. SOC hunts for that pattern in EDR telemetry and finds two infected machines that never triggered an alert.

**Example 3 — IR enrichment:**
During an incident, IR team finds a malware sample. CTI analyst identifies it as Emotet. This immediately tells the IR team: expect lateral movement via SMB, look for TrickBot secondary payload, check for banking credential theft.

---

### 5️⃣ Diagrams / Flow

```
CTI ←→ SOC INTEGRATION FLOW:

[CTI Team]
    │ Produces IOCs, TTPs, Actor Profiles
    ▼
[TIP (MISP/OpenCTI)]
    │ Normalizes and distributes intel
    ├──► [SIEM] → Alert enrichment, detection rules
    ├──► [EDR] → Behavioral detection tuning
    ├──► [SOAR] → Automated response playbooks
    └──► [IR Team] → Investigation context

[SOC/IR] → feeds back new IOCs/TTPs to [CTI Team]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- The **4 types of CTI** (Strategic, Operational, Tactical, Technical) and which teams use each
- **IOC enrichment** — adding actor/campaign context to raw indicators
- CTI role in each **IR phase** (especially Detection/Analysis and Containment)
- **TIP** as the central platform for CTI-SOC integration
- Difference between **reactive SOC** (alert-driven) and **proactive SOC** (CTI-driven hunting)

**Common traps:**
- ⚠️ Tactical CTI (IOCs) is the most perishable — IPs and domains change frequently
- ⚠️ Strategic CTI is NOT for SOC analysts — it's for CISOs and executives
- ⚠️ CTI integration requires a TIP — without it, IOC management is manual and error-prone

---

### 7️⃣ Quick Revision

- CTI transforms SOC from **reactive to proactive**
- **4 CTI types**: Strategic (exec), Operational (IR), Tactical (SIEM/blocking), Technical (malware analysts)
- CTI enriches alerts, guides threat hunting, and informs IR playbooks
- **TIP** (MISP, OpenCTI, Anomali) is the central hub for CTI-SOC integration
- **Lessons learned** in IR feeds new intel back to the CTI lifecycle

---

### 8️⃣ Practice Questions

**Q1.** Which type of CTI is most useful for a SOC analyst triaging alerts?
- A) Strategic
- B) Operational
- **C) Tactical** ✅
- D) Technical

**Q2.** A CTI analyst discovers that a threat actor uses a specific PowerShell obfuscation technique. The SOC uses this to proactively search endpoints. This activity is called:
- A) Incident response
- B) Alert triage
- **C) Threat hunting** ✅
- D) Vulnerability management

**Q3.** During which IR phase does CTI provide the most value by enriching alerts with actor context?
- A) Preparation
- **B) Detection and Analysis** ✅
- C) Recovery
- D) Lessons Learned

**Q4.** Which platform is used to manage, normalize, and distribute CTI to SOC tools?
- A) SIEM
- B) EDR
- **C) TIP (Threat Intelligence Platform)** ✅
- D) SOAR

**Q5.** What makes tactical CTI the most perishable type?
- A) It requires legal review before use
- B) It is only useful for executives
- **C) IOCs like IPs and domains change frequently as actors rotate infrastructure** ✅
- D) It requires a STIX format to be actionable

---

## 9. PIR / RFI Cycle

### 1️⃣ Core Concept
**PIR (Priority Intelligence Requirement)** and **RFI (Request for Intelligence)** are the formal mechanisms by which intelligence consumers tell the CTI team **what questions need to be answered**. They drive the entire CTI lifecycle — without them, analysts collect randomly and produce irrelevant intel.

- **PIR** = standing, recurring top-priority questions (set by leadership)
- **RFI** = ad-hoc, specific requests from any consumer (submitted on-demand)

---

### 2️⃣ Deep Dive

### PIR (Priority Intelligence Requirements)

- Set at the beginning of the planning cycle
- Defined by **senior leadership** (CISO, C-suite, board)
- Represent the **most critical intelligence gaps**
- Usually reviewed quarterly or after major incidents
- Limited in number — typically 3–7 PIRs at any time

**Examples of PIRs:**
- "Which threat actors are targeting our sector this quarter?"
- "What ransomware groups have active campaigns against financial institutions?"
- "Are there any known vulnerabilities in our critical infrastructure being actively exploited?"

### RFI (Request for Intelligence)

- Ad-hoc request from any intel consumer (SOC, IR team, CISO, vendor)
- Triggered by a specific event, incident, or question
- Has an associated **SLA** for response (e.g., 24 hours for P1 RFIs)
- Processed based on **priority and resource availability**

**RFI Process:**
1. Consumer submits RFI (question + context + deadline)
2. CTI team reviews and prioritizes
3. CTI team researches and produces response
4. Response delivered to requestor
5. Feedback collected for future improvement

### Intelligence Requirements Hierarchy:
```
CCIRs (Commander's Critical Info Requirements) — top level
  └── PIRs (Priority Intelligence Requirements) — leadership questions
        └── SIRs (Specific Intelligence Requirements) — sub-questions
              └── RFIs (ad-hoc requests from any consumer)
```

---

### 3️⃣ Key Terms & Definitions

- **PIR:** Priority Intelligence Requirement — top-priority recurring intelligence questions
- **RFI:** Request for Intelligence — ad-hoc intelligence request from a consumer
- **Intelligence Gap:** A question that CTI cannot currently answer with confidence
- **SLA:** Service Level Agreement — time commitment to respond to an RFI
- **CCIR:** Commander's Critical Information Requirement — highest-level requirement
- **SIR:** Specific Intelligence Requirement — detailed sub-question under a PIR
- **Consumer:** The recipient/user of CTI products (SOC, CISO, IR team, executives)
- **Collection Plan:** A plan outlining how each PIR/RFI will be answered (sources, methods)
- **Intelligence Requirement (IR):** General term for any CTI question or need

---

### 4️⃣ Examples

**Example 1 — PIR in action:**
A bank's CISO sets a PIR: "What financially-motivated APT groups are active against US banks?" The CTI team continuously monitors threat feeds, dark web, and ISAC reports to answer this ongoing question.

**Example 2 — RFI in action:**
An IR analyst finds a suspicious binary during an investigation. They submit an RFI: "What is this malware hash associated with? What actor uses it?" CTI responds within 4 hours with a full malware profile.

**Example 3 — PIR vs RFI:**
- PIR: "What is the current ransomware threat landscape for healthcare?" → Answered monthly in a standing report
- RFI: "Was this specific hospital just mentioned on a ransomware leak site?" → Ad-hoc, answered within hours

---

### 5️⃣ Diagrams / Flow

```
PIR / RFI CYCLE:

[Leadership/Consumers]
        │
        │ Define PIRs (quarterly)
        │ Submit RFIs (ad-hoc)
        ▼
[CTI Planning Phase]
        │
        │ Builds Collection Plan
        ▼
[Collection & Analysis]
        │
        │ Produces intelligence products
        ▼
[Dissemination]
        │
        │ Delivers PIR reports (recurring)
        │ Delivers RFI responses (on-demand)
        ▼
[Consumer Feedback]
        │
        └─────────────────► Refines PIRs / New RFIs
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **PIR vs RFI** — know the difference clearly
- **PIRs are set by leadership** (CISO/executives); RFIs can come from anyone
- PIRs drive the **Planning phase** of the CTI lifecycle
- The **SLA concept** for RFI responses
- RFIs are **reactive**; PIRs are **proactive**

**Common traps:**
- ⚠️ RFI ≠ PIR — PIR is recurring/strategic; RFI is ad-hoc/tactical
- ⚠️ PIRs should be **limited in number** — having 50 PIRs defeats the purpose of prioritization
- ⚠️ Collection without PIRs = undirected collection = waste of analyst time

---

### 7️⃣ Quick Revision

- **PIR** = recurring leadership questions (set quarterly) — "What do we always need to know?"
- **RFI** = ad-hoc consumer request — "What do I need to know right now?"
- PIRs drive the **Planning phase** of the CTI lifecycle
- RFIs have **SLA deadlines** for response
- Without PIRs, CTI collection is **undirected and inefficient**

---

### 8️⃣ Practice Questions

**Q1.** Who is primarily responsible for defining Priority Intelligence Requirements (PIRs)?
- A) SOC analysts
- **B) Senior leadership such as the CISO** ✅
- C) Incident responders
- D) Threat intelligence vendors

**Q2.** An IR analyst needs urgent information about a suspicious domain discovered during an incident. What mechanism should they use?
- A) PIR
- **B) RFI** ✅
- C) TLP
- D) STIX bundle

**Q3.** What is the main difference between a PIR and an RFI?
- A) PIRs are technical; RFIs are strategic
- B) PIRs are submitted by analysts; RFIs by executives
- **C) PIRs are recurring, priority-driven requirements; RFIs are ad-hoc, on-demand requests** ✅
- D) PIRs use STIX format; RFIs use PDF

**Q4.** A CTI team has 45 defined PIRs. What is the likely problem?
- A) Too few analysts to handle the workload
- **B) Too many PIRs dilutes prioritization — only a few should be designated as top priorities** ✅
- C) The PIRs are not in STIX format
- D) PIRs should be set monthly, not quarterly

**Q5.** What document outlines how a CTI team will answer each PIR?
- A) Threat report
- B) STIX bundle
- **C) Collection plan** ✅
- D) Incident report

---

## 10. Intel Maturity Model

### 1️⃣ Core Concept
The **Intel Maturity Model** provides a framework to **assess how advanced and capable a CTI program is**. It helps organizations understand where they currently stand and what they need to do to improve. Programs typically progress from ad-hoc/reactive to fully optimized/proactive.

---

### 2️⃣ Deep Dive

Several maturity models exist. The most referenced in CTI is the **Gartner CTI Maturity Model** and variations used by SANS and Recorded Future. Most use **5 levels:**

| Level | Name | Description |
|---|---|---|
| **0** | Non-existent | No CTI capability; no structured collection or analysis |
| **1** | Initial/Ad-hoc | Reactive; no formal process; individuals act on instinct |
| **2** | Defined | Basic processes exist; some IOC collection; limited sharing |
| **3** | Managed | Repeatable processes; PIRs defined; TIP in use; some automation |
| **4** | Quantitatively Managed | Metrics tracked; intel products measured for quality/impact |
| **5** | Optimizing | Continuous improvement; predictive; integrated across organization |

**What separates mature from immature programs:**

| Immature (Level 1) | Mature (Level 4-5) |
|---|---|
| Reacts to incidents | Hunts proactively before incidents |
| Collects random IOCs | Collects based on PIRs |
| Manual, spreadsheet-based | Automated TIP, STIX/TAXII |
| No sharing partners | Active ISAC participation |
| No metrics | Measures MTTD, MTTR, intel accuracy |
| Tactical only | Strategic, operational, tactical, technical |

**Maturity assessment process:**
1. Evaluate current capabilities across dimensions (collection, analysis, sharing, governance)
2. Map findings to maturity levels
3. Identify gaps
4. Build roadmap to next level
5. Re-assess periodically

**Key dimensions assessed:**
- People (skilled analysts, roles defined)
- Process (lifecycle, PIRs, governance)
- Technology (TIP, SIEM integration, automation)
- Sharing (ISAC participation, STIX/TAXII usage)

---

### 3️⃣ Key Terms & Definitions

- **Maturity Model:** Framework for assessing and improving program capability over time
- **Ad-hoc:** Unplanned, reactive, inconsistent — characteristic of Level 1
- **MTTD:** Mean Time to Detect — metric measuring detection speed
- **MTTR:** Mean Time to Respond — metric measuring response speed
- **Intel Quality Metrics:** Measurements of intel accuracy, relevance, timeliness
- **Capability Gap:** Area where current maturity is below target level
- **Roadmap:** Documented plan for advancing maturity level
- **TIP:** Threat Intelligence Platform — indicator of Level 3+ maturity

---

### 4️⃣ Examples

**Example 1 — Level 1 organization:**
A small company has one security analyst who occasionally googles threat news and manually blocks IPs. No formal CTI process exists. No PIRs, no TIP, no sharing. → Level 1 (Initial)

**Example 2 — Level 3 organization:**
A medium enterprise has a 3-person CTI team, uses MISP as a TIP, subscribes to commercial feeds, has defined PIRs set by the CISO, shares IOCs with their ISAC. → Level 3 (Managed)

**Example 3 — Level 5 organization:**
A large bank with a fully staffed CTI team, automated STIX/TAXII pipelines, measures intel accuracy rates and MTTD, produces strategic reports for the board, runs predictive threat modeling. → Level 5 (Optimizing)

---

### 5️⃣ Diagrams / Flow

```
CTI MATURITY PROGRESSION:

Level 0: No CTI ──► Level 1: Ad-hoc ──► Level 2: Defined
                                              │
                                              ▼
                         Level 5: Optimizing ◄── Level 4: Quantitative ◄── Level 3: Managed

Key Transitions:
L1→L2: Document processes, start collecting intentionally
L2→L3: Implement TIP, define PIRs, begin sharing
L3→L4: Add metrics, measure quality and impact
L4→L5: Predictive analytics, continuous improvement loop
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **The 5 maturity levels** (0-5) and their characteristics
- What distinguishes each level from the next
- A **TIP** is a sign of at least Level 3 maturity
- **Metrics** (MTTD, MTTR) appear at Level 4+
- The **goal** of a maturity model is improvement, not perfection

**Common traps:**
- ⚠️ A Level 2 program isn't "bad" — it's a milestone, not a failure
- ⚠️ Having a TIP doesn't automatically mean Level 5 — process and metrics matter too
- ⚠️ Maturity is measured across people, process, AND technology

---

### 7️⃣ Quick Revision

- Maturity models assess CTI programs on a **0–5 scale**
- Level 1 = reactive/ad-hoc; Level 5 = optimizing/predictive
- Key transitions: Process (L2), TIP adoption (L3), Metrics (L4), Predictive (L5)
- Assessed across **People, Process, Technology, and Sharing**
- Goal: Identify gaps and build a roadmap to improve

---

### 8️⃣ Practice Questions

**Q1.** An organization reacts to threats only after incidents occur and has no formal CTI process. What maturity level is this?
- A) Level 0
- **B) Level 1** ✅
- C) Level 2
- D) Level 3

**Q2.** Which capability most clearly indicates a CTI program has reached Level 3 maturity?
- A) The CISO reads threat news daily
- B) A single analyst manually blocks IPs
- **C) A Threat Intelligence Platform is implemented with PIRs defined** ✅
- D) The organization shares intel publicly

**Q3.** What metrics are most associated with Level 4 CTI maturity?
- **A) MTTD and MTTR tracking, intel accuracy measurement** ✅
- B) Number of IOCs collected
- C) Number of STIX bundles shared
- D) Cost of the CTI platform

**Q4.** A CTI program assessment reveals gaps in sharing with external partners and no defined PIRs. What is the recommended next step?
- A) Purchase a more expensive TIP
- B) Hire more analysts immediately
- **C) Build a maturity roadmap addressing PIR definition and ISAC participation** ✅
- D) Implement GDPR compliance first

**Q5.** Which three dimensions are assessed in a CTI maturity evaluation?
- A) Budget, headcount, and location
- B) Threat actors, IOCs, and reports
- **C) People, Process, and Technology** ✅
- D) SIEM, SOAR, and EDR

---

## 11. Executive Reporting

### 1️⃣ Core Concept
**Executive reporting** in CTI is the process of translating complex technical threat intelligence into **clear, concise, decision-ready information for senior leadership** (CEO, CISO, Board). Executives don't read IOC lists — they need **business risk context, implications, and recommendations**.

The goal: enable executives to make **informed, timely, risk-based decisions**.

---

### 2️⃣ Deep Dive

**Key principles of executive CTI reporting:**

1. **Know your audience** — Executives care about business risk, not technical details
2. **Lead with impact** — Start with "what does this mean for us?" not "here's how the malware works"
3. **Use clear language** — No jargon, acronyms unexplained, or overly technical terms
4. **Be concise** — Executive reports: 1–2 pages max (plus appendix for technical details)
5. **Actionable recommendations** — Always end with "what we recommend you approve/fund"
6. **Quantify risk** — Use business language: potential financial loss, regulatory exposure, reputational impact

**Types of executive CTI products:**

| Product | Audience | Frequency | Content |
|---|---|---|---|
| **Strategic Threat Briefing** | CISO, C-suite, Board | Monthly/Quarterly | Sector threats, actor profiles, trends |
| **Flash Report** | CISO, IR lead | As needed | Breaking threat, immediate risk assessment |
| **Threat Landscape Report** | Executive team | Quarterly | Annual threat trends, industry comparison |
| **Vulnerability Risk Brief** | CISO, CTO | As needed | Critical CVE + exploitation likelihood |
| **Incident Executive Summary** | CEO, Board | Post-incident | What happened, impact, response, lessons |

**Executive Report Structure (standard):**
1. **Executive Summary** (3–5 sentences) — the most important thing they need to know
2. **Key Findings** — 3–5 bullet points
3. **Business Impact** — risk in business terms
4. **Recommendations** — specific, prioritized actions
5. **Supporting Detail** (appendix) — technical data for those who want it

**Common mistakes:**
- Starting with technical details instead of business impact
- Using unexplained acronyms (MITRE, APT, TTP)
- Reporting threats without recommending actions
- Making the report too long

---

### 3️⃣ Key Terms & Definitions

- **Strategic Intelligence:** High-level, long-term threat landscape information for executives
- **Executive Brief:** Short, impact-focused intelligence product for senior leadership
- **Flash Report:** Urgent, short-notice report on an emerging or breaking threat
- **Threat Landscape:** The overall picture of threats facing an organization or sector
- **Business Risk:** Potential impact expressed in financial, operational, or reputational terms
- **Actionable Recommendation:** A specific, feasible action the executive can approve
- **TLP:AMBER/RED:** Often applied to executive reports to control distribution
- **Risk Appetite:** How much risk an organization is willing to accept

---

### 4️⃣ Examples

**Example — Technical vs Executive language:**

Technical (bad for exec): *"APT29 is leveraging T1059.001 with base64-encoded PowerShell payloads to establish C2 via T1071 over HTTPS to exfiltrate data."*

Executive (good): *"A Russian state-sponsored group is actively targeting organizations like ours using phishing emails to gain access to company systems. If successful, they could steal sensitive customer data, resulting in regulatory fines and reputational damage. We recommend approving budget for enhanced email security controls."*

**Example — Flash Report trigger:**
- CISA releases alert about active exploitation of a critical firewall vulnerability
- CTI team sends a 1-page flash to CISO within 2 hours: "This affects our infrastructure. Risk = HIGH. We recommend emergency patching within 24 hours."

---

### 5️⃣ Diagrams / Flow

```
EXECUTIVE REPORT WRITING PROCESS:

[Threat Intelligence Analysis]
        │
        ▼
[Translate Technical → Business Impact]
        │ (What does this mean for our revenue, data, compliance?)
        ▼
[Draft Executive Report]
        │ Executive Summary → Key Findings → Risk → Recommendations
        ▼
[Peer Review / Senior Analyst Check]
        │
        ▼
[TLP Classification + Distribution]
        │
        ▼
[Delivery to Executive Audience]
        │
        ▼
[Feedback → Improve next report]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- Executive reports focus on **business risk and recommendations**, not technical IOCs
- **Flash reports** are for urgent/breaking threats
- The correct structure: Executive Summary → Findings → Risk → Recommendations
- Executive reports should be **concise** (1–2 pages)
- Translate technical jargon into **business language**

**Common traps:**
- ⚠️ Don't confuse strategic CTI (for executives) with tactical CTI (for SOC)
- ⚠️ A good exec report always includes **recommendations** — without them, it's just information
- ⚠️ TLP is still applied to executive reports — they are NOT automatically public

---

### 7️⃣ Quick Revision

- Executive reports = **business risk + recommendations** in plain language
- **Flash report** = urgent, breaking threat notification
- Report structure: Summary → Findings → Risk → **Recommendations**
- Keep it to **1–2 pages** — technical details go in appendix
- Always translate TTPs and IOCs into **financial/operational risk language**

---

### 8️⃣ Practice Questions

**Q1.** A CTI analyst is writing a report for the CEO about a ransomware threat. Which content should come first?
- A) A list of malware IOCs
- B) ATT&CK technique mappings
- **C) An executive summary explaining business risk and impact** ✅
- D) Full technical malware analysis

**Q2.** A critical zero-day is actively being exploited. The CTI team needs to notify the CISO immediately with a concise risk assessment. What report type is appropriate?
- **A) Flash report** ✅
- B) Quarterly threat landscape report
- C) STIX bundle
- D) Incident post-mortem

**Q3.** Which of the following best exemplifies executive-level CTI communication?
- A) "T1566.001 was used for initial access via phishing attachment"
- **B) "A financially-motivated group is targeting our sector via phishing, potentially exposing customer data worth $5M in regulatory risk"** ✅
- C) "The C2 beaconed every 30 seconds using HTTP POST to 185.220.x.x"
- D) "EDR telemetry shows process injection via CreateRemoteThread API"

**Q4.** What is always required in an actionable executive CTI report?
- A) STIX-formatted IOCs
- B) Kill Chain mapping
- **C) Specific, prioritized recommendations for decision-makers** ✅
- D) Full malware code analysis

**Q5.** How long should a standard executive CTI brief typically be?
- A) 10–20 pages with full technical details
- **B) 1–2 pages with technical details in an appendix** ✅
- C) A single paragraph
- D) No limit — include all findings

---

## 12. Compliance Alignment

### 1️⃣ Core Concept
**Compliance alignment** in CTI means ensuring that the threat intelligence program is aware of, and contributes to, the organization's **regulatory and framework compliance requirements** (NIST, ISO 27001, GDPR, etc.). CTI doesn't operate in isolation — it must support the broader compliance and risk management ecosystem.

---

### 2️⃣ Deep Dive

**Key frameworks relevant to CTI:**

| Framework | Relevance to CTI |
|---|---|
| **NIST CSF** | CTI maps to "Identify" and "Detect" functions; supports the full framework |
| **NIST SP 800-150** | Specific guide for CTI sharing in government |
| **ISO/IEC 27001** | CTI supports threat assessment for ISMS |
| **GDPR / DPDP** | CTI collection must respect personal data laws |
| **DORA (EU)** | Financial sector must demonstrate threat intelligence capability |
| **NIS2 (EU)** | Requires threat intelligence as part of cybersecurity obligations |
| **PCI DSS** | CTI supports threat monitoring requirements |
| **HIPAA** | CTI helps protect healthcare PHI from targeted threats |

**NIST Cybersecurity Framework (CSF) and CTI:**

| CSF Function | CTI Role |
|---|---|
| **Identify** | Asset inventory, threat landscape, risk assessment |
| **Protect** | Inform security controls based on threat actors |
| **Detect** | IOC-driven SIEM rules, threat hunting |
| **Respond** | CTI supports IR with actor context |
| **Recover** | CTI identifies if actor may return, alternative entry points |

**Compliance requirements CTI typically supports:**
- Threat and vulnerability management (ISO 27001 Annex A.12.6)
- Information sharing with authorities/regulators (NIS2)
- Evidence of monitoring and detection (PCI DSS Req 10-11)
- Risk assessment documentation (NIST CSF "Identify")

**CTI governance for compliance:**
- Document all collection methods and sources
- Maintain audit trails of intel received and shared
- Apply appropriate data classification and TLP
- Ensure GDPR compliance when processing personal data of threat actors

---

### 3️⃣ Key Terms & Definitions

- **NIST CSF:** National Institute of Standards and Technology Cybersecurity Framework — 5 functions: Identify, Protect, Detect, Respond, Recover
- **ISO 27001:** International standard for Information Security Management Systems (ISMS)
- **DORA:** Digital Operational Resilience Act — EU financial sector cybersecurity regulation
- **NIS2:** Network and Information Security Directive 2 — EU cybersecurity obligation for critical sectors
- **ISMS:** Information Security Management System — systematic approach to managing information security
- **Compliance:** Adherence to laws, regulations, standards, and frameworks
- **Audit Trail:** Documented record of activities for compliance verification
- **Risk Assessment:** Identifying, analyzing, and evaluating cybersecurity risks

---

### 4️⃣ Examples

**Example 1 — NIST CSF:**
A company maps CTI activities to NIST CSF during their annual assessment. CTI's IOC feeds feed into "Detect"; threat landscape reports feed "Identify" — demonstrating CTI's contribution to the framework.

**Example 2 — GDPR concern:**
A CTI analyst wants to share a threat actor's personal details (real name, address). Under GDPR, this personal data cannot be freely shared without a lawful basis — legal must be consulted.

**Example 3 — DORA compliance:**
A European bank must demonstrate CTI capability to regulators. They show: ISAC membership, automated STIX/TAXII feeds, quarterly threat landscape reports to the board — all documented.

---

### 5️⃣ Diagrams / Flow

```
CTI SUPPORTING NIST CSF:

IDENTIFY → CTI provides threat landscape, actor profiles, asset risk context
     │
PROTECT → CTI informs security control selection based on known adversary TTPs
     │
DETECT → CTI-driven SIEM rules and IOC feeds trigger alerts
     │
RESPOND → CTI provides IR teams with actor attribution and behavioral context
     │
RECOVER → CTI identifies persistence mechanisms, alternate access, re-infection risk
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **NIST CSF 5 functions** and how CTI maps to them (especially Identify and Detect)
- **NIST SP 800-150** — the CTI-specific NIST guide
- **ISO 27001** as an ISMS standard
- **GDPR constraints** on collecting and sharing personal data in CTI
- **DORA and NIS2** as EU-specific compliance obligations for CTI

**Common traps:**
- ⚠️ NIST CSF is a **framework** (voluntary guidance), not a regulation
- ⚠️ ISO 27001 is a **certifiable standard** — organizations can be certified
- ⚠️ GDPR applies to **personal data** — threat actor personal info may be covered

---

### 7️⃣ Quick Revision

- CTI supports compliance frameworks: **NIST CSF, ISO 27001, DORA, NIS2, PCI DSS**
- NIST CSF maps CTI to: **Identify** (threat landscape) and **Detect** (IOC feeds) primarily
- **NIST SP 800-150** = specific guide for CTI sharing
- **GDPR** constrains collection of personal data even on threat actors
- Document and audit-trail all CTI activities for compliance evidence

---

### 8️⃣ Practice Questions

**Q1.** Which NIST CSF function does CTI most directly support when providing IOC-driven detection rules to the SIEM?
- A) Identify
- B) Protect
- **C) Detect** ✅
- D) Recover

**Q2.** Which NIST publication specifically addresses cyber threat intelligence sharing?
- A) NIST SP 800-53
- B) NIST SP 800-61
- **C) NIST SP 800-150** ✅
- D) NIST SP 800-171

**Q3.** A CTI analyst wants to share a threat actor's personal information with a partner organization. Which regulation primarily constrains this activity in Europe?
- **A) GDPR** ✅
- B) HIPAA
- C) PCI DSS
- D) SOX

**Q4.** ISO 27001 is best described as:
- A) A US government cybersecurity regulation
- **B) An international, certifiable standard for Information Security Management Systems** ✅
- C) A threat intelligence sharing protocol
- D) A framework specific to financial institutions

**Q5.** The EU Digital Operational Resilience Act (DORA) is primarily targeted at which sector?
- A) Healthcare
- B) Energy
- **C) Financial services** ✅
- D) Transportation

---

# PHASE 3 — APPLIED PRACTICE

---

## 13. Reading APT Reports

### 1️⃣ Core Concept
**APT (Advanced Persistent Threat) reports** are detailed intelligence products produced by threat intelligence vendors (Mandiant, CrowdStrike, CISA, Microsoft MSTIC) that document the activities, tools, and TTPs of specific threat actors. CTI analysts must be able to **efficiently extract actionable intelligence** from these reports.

---

### 2️⃣ Deep Dive

**What APT reports typically contain:**

| Section | What to Look For |
|---|---|
| **Executive Summary** | Actor overview, targeted sectors, key findings |
| **Actor Profile** | Nation-state or financially motivated? Aliases? History? |
| **Initial Access Methods** | How they get in (phishing, exploit, supply chain) |
| **TTPs** | MITRE ATT&CK technique IDs used |
| **Malware/Tools Used** | Names, capabilities, C2 methods |
| **Infrastructure** | IPs, domains, hosting providers |
| **IOCs** | Hashes, IPs, domains, URLs, email addresses |
| **Detection Guidance** | YARA rules, Sigma rules, hunting queries |
| **Mitigations** | Recommended defensive actions |

**Key APT report publishers:**
- **Mandiant/Google:** APT1, APT28, APT41, etc. — highly detailed
- **CrowdStrike:** Uses animal naming (Fancy Bear, Cozy Bear, Lazarus) — incident-driven
- **CISA Advisories:** Government-vetted, often joint (FBI + CISA + NSA)
- **Microsoft MSTIC:** Often focused on Azure/M365 targeting
- **Recorded Future, Secureworks, Palo Alto Unit42:** Commercial vendors

**How to read efficiently:**
1. Read **Executive Summary** first — decide if relevant
2. Check **actor profile** — do they target your sector/region/technology?
3. Extract **IOCs** for immediate SIEM/TIP ingestion
4. Map **TTPs** to MITRE ATT&CK — identify detection gaps
5. Review **detection guidance** — implement YARA/Sigma rules
6. Read **full report** only if actor is high relevance

**Named APT groups to know:**
- **APT28 / Fancy Bear:** Russian GRU; targets NATO, governments, elections
- **APT29 / Cozy Bear:** Russian SVR; espionage, supply chain (SolarWinds)
- **APT41 / Winnti:** Chinese MSS; espionage + cybercrime
- **Lazarus Group:** North Korean DPRK; financial theft, ransomware
- **FIN7 / Carbanak:** Financially motivated; targeting retail/hospitality/finance

---

### 3️⃣ Key Terms & Definitions

- **APT:** Advanced Persistent Threat — highly skilled, well-resourced threat actor (often nation-state)
- **IOC (Indicator of Compromise):** Technical artifact indicating a breach or malicious activity
- **YARA Rule:** Pattern-matching rule used to identify malware samples
- **Sigma Rule:** SIEM-agnostic detection rule format
- **C2 (Command & Control):** Infrastructure used to remotely control malware
- **Attribution:** Identifying the actor behind an attack
- **Alias:** Alternative names for the same threat group (e.g., APT28 = Fancy Bear = Sofacy)
- **MSTIC:** Microsoft Threat Intelligence Center
- **TTPs:** Tactics, Techniques, and Procedures — behavioral fingerprint of a threat actor

---

### 4️⃣ Examples

**Example — Reading a Mandiant APT28 report:**
1. Executive summary: Russian GRU targeting Ukrainian government and NATO allies
2. Check relevance: Our org is a NATO contractor → HIGH relevance
3. Extract IOCs: 15 C2 domains, 4 file hashes → ingest into MISP
4. Map TTPs: T1566 (phishing), T1059 (PowerShell), T1027 (obfuscation)
5. Detection gaps: No detection for T1027 → create Sigma rule
6. Implement mitigation: Enable MFA, deploy email gateway rules

---

### 5️⃣ Diagrams / Flow

```
APT REPORT READING WORKFLOW:

[Receive APT Report]
        │
        ▼
[Read Executive Summary]
        │
        ├── Not Relevant → File/Archive
        │
        └── Relevant → Continue
                │
                ▼
        [Extract IOCs] → Ingest to SIEM/TIP
                │
                ▼
        [Map TTPs to ATT&CK] → Find detection gaps
                │
                ▼
        [Implement YARA/Sigma rules]
                │
                ▼
        [Brief SOC/IR team]
                │
                ▼
        [Produce PIR/RFI response if applicable]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- Key APT groups and their affiliations (APT28=Russia, Lazarus=DPRK, APT41=China)
- How to **prioritize reading** an APT report efficiently
- What **YARA and Sigma rules** are used for
- Major APT report publishers: Mandiant, CrowdStrike, CISA
- How to connect APT report findings to MITRE ATT&CK

**Common traps:**
- ⚠️ APT doesn't always mean nation-state — FIN7 is APT-level capability but financially motivated
- ⚠️ IOCs from old reports may be stale — check dates before ingesting
- ⚠️ Don't ingest all IOCs blindly — validate relevance first

---

### 7️⃣ Quick Revision

- APT reports = detailed intelligence on threat actor TTPs, IOCs, tools, and mitigations
- Key publishers: **Mandiant, CrowdStrike, CISA, Microsoft MSTIC**
- Read order: Executive Summary → Relevance → IOCs → TTPs → Detection rules
- Key groups: APT28 (Russia), APT29 (Russia), APT41 (China), Lazarus (North Korea), FIN7 (criminal)
- **YARA** = malware detection; **Sigma** = SIEM detection rule format

---

### 8️⃣ Practice Questions

**Q1.** Which nation-state is APT29 (Cozy Bear) associated with?
- A) China
- B) North Korea
- **C) Russia** ✅
- D) Iran

**Q2.** A CTI analyst receives a new APT report. What should they check first to determine relevance?
- A) IOC list
- **B) Executive summary — actor profile and targeted sectors** ✅
- C) Full technical malware analysis
- D) YARA rules section

**Q3.** What is a YARA rule used for?
- A) Transporting STIX bundles
- B) Classifying intel sharing levels
- **C) Pattern-matching to identify malware samples** ✅
- D) Defining intelligence requirements

**Q4.** Lazarus Group is primarily attributed to which nation-state?
- A) Russia
- B) China
- C) Iran
- **D) North Korea** ✅

**Q5.** After extracting IOCs from an APT report, what should an analyst do BEFORE ingesting them into the SIEM?
- A) Convert them to PDF format
- B) Immediately block all associated IPs
- **C) Validate their relevance and check the report date for staleness** ✅
- D) Send them to the CISO for approval

---

## 14. MITRE ATT&CK CTI Training

### 1️⃣ Core Concept
MITRE provides **free, official CTI training** at **attack.mitre.org/resources** that teaches analysts how to use ATT&CK for threat intelligence purposes — from mapping actor behaviors to creating ATT&CK-based reports and using ATT&CK Navigator.

---

### 2️⃣ Deep Dive

**What the MITRE ATT&CK CTI training covers:**

| Module | Content |
|---|---|
| **ATT&CK for CTI** | How to use ATT&CK to describe and analyze threat actor behavior |
| **Mapping to ATT&CK** | How to read a threat report and map it to ATT&CK techniques |
| **ATT&CK Navigator** | How to use the web-based visualization tool |
| **Threat Group Profiles** | How ATT&CK tracks and documents threat actors |
| **Detection with ATT&CK** | Using ATT&CK to improve detection coverage |

**ATT&CK CTI Use Cases taught:**
1. **Describe threat actors** using ATT&CK vocabulary
2. **Prioritize defenses** based on which techniques actors use
3. **Compare threat groups** to identify overlaps
4. **Track campaigns** over time using ATT&CK Navigator layers
5. **Report intel** using standardized ATT&CK technique IDs

**ATT&CK Navigator — key features:**
- Web-based, free tool (mitre-attack.github.io/attack-navigator)
- Create layers showing which techniques an actor uses
- Color-code by detection coverage, actor activity, or threat priority
- Compare multiple layers (e.g., what techniques APT28 uses vs what you detect)
- Export as JSON or SVG

**How to map a report to ATT&CK:**
1. Read threat report, find behavioral descriptions
2. Match behaviors to ATT&CK technique descriptions
3. Assign technique IDs (e.g., "sent phishing email" → T1566.001)
4. Build ATT&CK Navigator layer
5. Identify detection gaps (techniques used but not detected)

---

### 3️⃣ Key Terms & Definitions

- **ATT&CK Navigator:** Free web tool for visualizing ATT&CK technique coverage
- **Navigator Layer:** A saved configuration in ATT&CK Navigator showing technique selections
- **Technique Mapping:** The process of connecting observed behavior to ATT&CK technique IDs
- **Detection Coverage:** Which ATT&CK techniques your security tools can detect
- **ATT&CK for CTI:** MITRE's specific guidance on using ATT&CK in CTI workflows
- **Threat Group:** A named adversary tracked in ATT&CK (e.g., G0007 = APT28)
- **Software:** ATT&CK term for malware and tools (e.g., S0002 = Mimikatz)
- **Group ID:** ATT&CK unique identifier for threat actors (e.g., G0016 = APT29)

---

### 4️⃣ Examples

**Example — Mapping a phishing attack to ATT&CK:**
- Report says: "Attackers sent a Word document with a malicious macro via email"
- Mapping:
  - T1566.001 — Phishing: Spearphishing Attachment
  - T1204.002 — User Execution: Malicious File
  - T1059.005 — Command and Scripting Interpreter: Visual Basic

**Example — ATT&CK Navigator gap analysis:**
- Layer 1: APT28 techniques (red = used by actor)
- Layer 2: Our detection coverage (green = we detect)
- Compare: Techniques that are red but not green = **gaps to address**

---

### 5️⃣ Diagrams / Flow

```
ATT&CK CTI WORKFLOW:

[Read Threat Report / Observe Incident]
        │
        ▼
[Extract Behaviors] (e.g., "PowerShell was used to download payload")
        │
        ▼
[Search ATT&CK for matching technique]
        │ → T1059.001 – PowerShell
        ▼
[Document in Navigator Layer]
        │
        ▼
[Overlay with Detection Coverage Layer]
        │
        ▼
[Identify Gaps] → Create detection rules, adjust controls
        │
        ▼
[Include ATT&CK IDs in CTI reports]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- ATT&CK Navigator is a **free tool** — available at mitre-attack.github.io
- **Technique mapping** process — behavior → ATT&CK technique ID
- ATT&CK Group IDs (G0007, G0016) and Software IDs (S0002)
- Using Navigator layers to perform **detection gap analysis**
- ATT&CK training is available free at **attack.mitre.org/resources**

**Common traps:**
- ⚠️ ATT&CK mapping is **interpretive** — one behavior can map to multiple techniques
- ⚠️ Navigator does NOT automatically detect threats — it's a **visualization/planning tool**
- ⚠️ ATT&CK Threat Groups use G-IDs, not APT numbers from other vendors

---

### 7️⃣ Quick Revision

- MITRE offers **free CTI training** at attack.mitre.org/resources
- **ATT&CK Navigator** = free web tool for visualizing technique coverage
- Technique mapping: observed behavior → ATT&CK technique ID
- Navigator layers enable **detection gap analysis** by comparing actor TTPs vs your coverage
- Threat groups have **G-IDs**; malware/tools have **S-IDs** in ATT&CK

---

### 8️⃣ Practice Questions

**Q1.** Where can analysts find MITRE's free ATT&CK CTI training?
- A) sans.org/for578
- **B) attack.mitre.org/resources** ✅
- C) cisa.gov/training
- D) nist.gov/ctf

**Q2.** An analyst reads "attackers used PowerShell to download and execute payloads." What ATT&CK technique best maps to this?
- A) T1566.001
- **B) T1059.001** ✅
- C) T1078
- D) T1021.001

**Q3.** What is the primary use of ATT&CK Navigator?
- A) Automatically blocking attack techniques in real-time
- **B) Visualizing and analyzing ATT&CK technique coverage and gaps** ✅
- C) Generating STIX bundles for sharing
- D) Running automated penetration tests

**Q4.** In ATT&CK, what does a "G-ID" represent?
- A) A specific malware sample
- **B) A named threat group** ✅
- C) A governance framework
- D) A detection rule set

**Q5.** When comparing two ATT&CK Navigator layers — one showing APT techniques and one showing detection coverage — what does an uncolored (gap) technique indicate?
- A) The technique is not real
- **B) The technique is used by the actor but not detected by current controls** ✅
- C) The technique is only used in mobile attacks
- D) The technique is deprecated

---

## 15. OpenCTI / MISP

### 1️⃣ Core Concept
**OpenCTI** and **MISP** are two of the most widely used **open-source Threat Intelligence Platforms (TIPs)**. They help CTI teams collect, store, manage, enrich, correlate, and share threat intelligence at scale. Both are free and widely deployed in government, enterprise, and security community environments.

---

### 2️⃣ Deep Dive

### MISP (Malware Information Sharing Platform)

- Created by **CIRCL (Computer Incident Response Center Luxembourg)**
- Originally focused on **IOC sharing** — IPs, domains, hashes
- Now supports **STIX 2.1, ATT&CK mappings, threat actor profiles**
- Used heavily by **ISACs, CERTs, and national cybersecurity agencies**
- Supports **correlation** — automatically links related events
- Has a **REST API** for integration with SIEM, SOAR, EDR

**MISP Key Features:**
- Events: Collections of related IOCs and attributes
- Objects: Structured data (e.g., file object = hash + filename + size)
- Tags: Classify data (TLP, sector, threat type)
- Galaxies: Pre-built collections (MITRE ATT&CK, threat actors, ransomware families)
- Feeds: Automatic ingestion from external sources
- Sharing Groups: Control who sees what data

---

### OpenCTI

- Created by **Filigran** (originally ANSSI-supported)
- More **analyst-focused** — graph-based visualization of relationships
- Native **STIX 2.1** support throughout
- Better for **complex threat actor profiling and campaign analysis**
- Integrates with MISP, MITRE ATT&CK, threat feeds
- Has a built-in **investigation workbench**

**OpenCTI Key Features:**
- Entities: Threat actors, campaigns, malware, indicators, vulnerabilities
- Relationships: Visual graph showing connections between entities
- Connectors: Ingest from MISP, VirusTotal, Shodan, AlienVault, etc.
- Reports: Store finished intelligence linked to entities
- Workspaces: Analyst investigation boards

---

**MISP vs OpenCTI:**

| Feature | MISP | OpenCTI |
|---|---|---|
| Primary focus | IOC sharing | Threat actor profiling & analysis |
| Data model | Events + Attributes | STIX 2.1 entities and relationships |
| Visualization | Basic | Advanced graph visualization |
| Sharing | Strong (sharing groups, feeds) | Via STIX/TAXII connectors |
| Community | Large, established | Growing, more modern |
| Best for | ISACs, rapid IOC sharing | Enterprise CTI teams, deep analysis |

---

### 3️⃣ Key Terms & Definitions

- **TIP:** Threat Intelligence Platform — manages, normalizes, and distributes CTI
- **MISP Event:** A collection of related IOCs and attributes in MISP
- **MISP Galaxy:** Pre-built threat actor/group/malware database in MISP
- **OpenCTI Connector:** Plugin that ingests data from external sources
- **CIRCL:** Computer Incident Response Center Luxembourg — creator of MISP
- **Correlation:** Automatically linking related IOCs or events
- **Sharing Group:** MISP feature controlling which organizations see which data
- **STIX 2.1:** Native data format for OpenCTI
- **Feed:** Automated ingestion of external threat intelligence into a TIP

---

### 4️⃣ Examples

**Example 1 — MISP use case:**
A national CERT ingests IOCs from 50 member organizations into MISP. MISP automatically correlates: same C2 IP appearing in events from 5 different members = likely coordinated campaign. Alert sent to all members.

**Example 2 — OpenCTI use case:**
A CTI analyst is profiling APT28. In OpenCTI:
- Creates a Threat Actor entity for APT28
- Links it to: malware used, campaigns, targeted sectors, TTPs
- Visual graph shows relationships
- Generates a finished intelligence report linked to all entities

**Example 3 — Integration:**
MISP → pushes IOCs via STIX/TAXII → OpenCTI → enriches with VirusTotal connector → sends to SIEM for automated detection

---

### 5️⃣ Diagrams / Flow

```
CTI PLATFORM ECOSYSTEM:

[External Feeds] ──► [MISP / OpenCTI (TIP)]
                              │
                   ┌──────────┼──────────┐
                   ▼          ▼          ▼
               [SIEM]      [SOAR]     [EDR]
            (detection)  (response) (endpoint)
                   │
                   ▼
              [SOC Analyst]
                   │
                   ▼
              [CTI Analyst] ──► [Enrichment] ──► [Reports]
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **MISP** = IOC sharing focus; **OpenCTI** = analysis/visualization focus
- MISP was created by **CIRCL**
- Both support **STIX 2.1**
- MISP **Galaxies** = pre-built threat intelligence collections (ATT&CK, actors)
- OpenCTI has **connectors** for external data ingestion

**Common traps:**
- ⚠️ MISP ≠ OpenCTI — they serve different primary purposes
- ⚠️ Both are **open-source and free** — no cost to use
- ⚠️ A "MISP Event" is not an "incident" — it's a collection of related IOCs

---

### 7️⃣ Quick Revision

- **MISP** = open-source TIP focused on **IOC sharing**; created by CIRCL
- **OpenCTI** = open-source TIP focused on **threat analysis and visualization**; STIX-native
- Both support **STIX 2.1** and integrate with SIEM/SOAR/EDR
- MISP Galaxies = pre-built ATT&CK/actor/malware data
- OpenCTI Connectors = ingest from VirusTotal, Shodan, MISP, etc.

---

### 8️⃣ Practice Questions

**Q1.** Which organization created MISP?
- A) MITRE
- **B) CIRCL** ✅
- C) Filigran
- D) SANS Institute

**Q2.** A CTI team needs to visually map relationships between threat actors, campaigns, and malware families. Which TIP is better suited?
- A) MISP
- **B) OpenCTI** ✅
- C) Splunk
- D) QRadar

**Q3.** What is a MISP Galaxy?
- A) A visualization tool for ATT&CK
- **B) A pre-built collection of threat intelligence categories (actors, malware, ATT&CK)** ✅
- C) A TAXII server for external sharing
- D) A report template format

**Q4.** Both MISP and OpenCTI natively support which threat intelligence format?
- A) XML
- **B) STIX 2.1** ✅
- C) CSV
- D) PDF

**Q5.** An OpenCTI "connector" is best described as:
- A) A TLP marking applied to reports
- B) A sharing group for ISAC members
- **C) A plugin that ingests data from external sources (e.g., VirusTotal, Shodan)** ✅
- D) A YARA rule deployment mechanism

---

## 16. SANS FOR578

### 1️⃣ Core Concept
**SANS FOR578** (Cyber Threat Intelligence) is a professional training course from the SANS Institute that covers the full spectrum of CTI skills — from analytical frameworks to collection, analysis, reporting, and platform usage. It is the **most recognized CTI-specific training course** in the industry and is aligned with many CTI certification requirements.

---

### 2️⃣ Deep Dive

**What FOR578 covers:**

| Module Area | Topics |
|---|---|
| **Frameworks** | Kill Chain, Diamond Model, ATT&CK |
| **Collection** | OSINT, dark web, technical collection |
| **Analysis** | Structured Analytic Techniques (SATs) |
| **Sharing** | STIX/TAXII, TLP, ISAC participation |
| **Platforms** | MISP, OpenCTI |
| **Reporting** | Tactical, operational, strategic reports |
| **Attribution** | Methods, ethics, confidence levels |
| **APT Tracking** | Group tracking, campaign analysis |

**Key Structured Analytic Techniques (SATs) taught in FOR578:**
- **Analysis of Competing Hypotheses (ACH):** Systematically test multiple hypotheses against evidence — reduces bias
- **Indicators Development:** Creating new detection indicators from behavioral observations
- **Link Analysis:** Connecting entities (actors, IPs, domains, malware) to reveal relationships
- **Kill Chain Analysis:** Using the Kill Chain to understand adversary campaign stages
- **Diamond Model Analysis:** Event-based intrusion analysis

**FOR578 deliverables (what students produce):**
- Threat actor profile report
- Tactical intelligence report with IOCs
- Strategic threat landscape brief
- ATT&CK Navigator layer for an actor

**Free FOR578 resources:**
- SANS blog posts at sans.org/blog
- Sample course materials shared at conferences (DFIR Summit, CTI Summit)
- YouTube recordings of SANS webinars on CTI topics

---

### 3️⃣ Key Terms & Definitions

- **FOR578:** SANS course number for Cyber Threat Intelligence
- **SAT (Structured Analytic Technique):** A systematic method to reduce cognitive bias in analysis
- **ACH (Analysis of Competing Hypotheses):** Analytical technique testing multiple explanations against evidence
- **Cognitive Bias:** Mental shortcuts that can lead to flawed analysis (e.g., confirmation bias)
- **Confirmation Bias:** Favoring evidence that confirms existing beliefs — major threat to CTI quality
- **Link Analysis:** Visual technique for mapping relationships between intelligence entities
- **Attribution Confidence:** The degree of certainty assigned to a threat actor identification
- **GIAC GCTI:** The SANS/GIAC certification tied to FOR578 knowledge

---

### 4️⃣ Examples

**Example — ACH in practice:**
A CTI analyst observes a phishing campaign. Two hypotheses:
- H1: This is APT28 (Russian GRU)
- H2: This is a copycat criminal group

Evidence tested:
- C2 overlaps with known APT28 infrastructure → supports H1
- Targeting of financial sector → inconsistent with APT28 (supports H2)
- Use of Zebrocy malware → strongly supports H1

Result: H1 more likely, but not conclusive → "Medium confidence APT28"

**Example — Link Analysis:**
Analyst maps: phishing domain → resolves to IP → IP hosted malware → malware communicates with another domain → that domain linked to a known APT group infrastructure cluster.

---

### 5️⃣ Diagrams / Flow

```
STRUCTURED ANALYTIC TECHNIQUE — ACH PROCESS:

1. Identify all plausible hypotheses (H1, H2, H3...)
2. List all significant evidence
3. Test each piece of evidence against each hypothesis:
   Consistent (+), Inconsistent (-), Neutral (~)
4. Build matrix:

         | H1 (APT28) | H2 (Criminal) | H3 (Hacktivist) |
Evidence1|    +       |      -        |       ~         |
Evidence2|    +       |      +        |       -         |
Evidence3|    ~       |      -        |       +         |

5. The hypothesis with MOST INCONSISTENCIES = LEAST LIKELY
6. The hypothesis with fewest inconsistencies = MOST LIKELY
7. Assign confidence level
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- FOR578 covers **CTI frameworks, collection, analysis, reporting, and platforms**
- **ACH (Analysis of Competing Hypotheses)** — the premier SAT for CTI analysis
- **Confirmation bias** — the most common cognitive bias in CTI analysis
- **GIAC GCTI** = certification associated with FOR578
- Free resources available at sans.org

**Common traps:**
- ⚠️ ACH does NOT prove a hypothesis true — it identifies the **least unlikely** hypothesis
- ⚠️ FOR578 is a training course, not a free resource — but related blog content is free
- ⚠️ Cognitive bias affects all analysts — SATs are designed to systematically counter this

---

### 7️⃣ Quick Revision

- **SANS FOR578** = premier CTI training; covers frameworks, collection, analysis, sharing
- **ACH** = Structured Analytic Technique — tests hypotheses against evidence systematically
- **Confirmation bias** = most dangerous cognitive bias in CTI
- **GIAC GCTI** = certification tied to FOR578
- ACH identifies the **least unlikely** hypothesis, not the "correct" one

---

### 8️⃣ Practice Questions

**Q1.** What is the primary purpose of Structured Analytic Techniques (SATs) in CTI?
- A) Automate threat intelligence collection
- **B) Reduce cognitive bias in the analytical process** ✅
- C) Format intelligence reports in STIX
- D) Accelerate malware reverse engineering

**Q2.** In ACH, the most likely hypothesis is identified by:
- A) The highest number of supporting evidence items
- **B) The fewest inconsistencies across evidence** ✅
- C) Expert opinion
- D) Historical precedent

**Q3.** Which cognitive bias is most dangerous in CTI analysis?
- A) Anchoring bias
- **B) Confirmation bias** ✅
- C) Availability bias
- D) Bandwagon effect

**Q4.** Which GIAC certification is associated with SANS FOR578?
- A) GSEC
- B) GCFE
- **C) GCTI** ✅
- D) GREM

**Q5.** An analyst creates a matrix with hypotheses in columns and evidence in rows, marking each cell as consistent, inconsistent, or neutral. This is:
- **A) Analysis of Competing Hypotheses (ACH)** ✅
- B) Link analysis
- C) Kill Chain analysis
- D) Diamond Model mapping

---

## 17. F3EAD Cycle

### 1️⃣ Core Concept
The **F3EAD cycle** (Find, Fix, Finish, Exploit, Analyze, Disseminate) is an intelligence-driven targeting cycle originally from military special operations. In cybersecurity CTI, it is used as an **intelligence-led operations model** that closely integrates threat intelligence with active response and hunting — more operationally focused than the standard CTI lifecycle.

---

### 2️⃣ Deep Dive

**The 6 phases of F3EAD:**

| Phase | Military Meaning | CTI/Cyber Meaning |
|---|---|---|
| **Find** | Locate the target | Identify the threat actor / malicious infrastructure |
| **Fix** | Pin down the target's location | Track and monitor the actor's activities continuously |
| **Finish** | Engage and neutralize | Take action — incident response, blocking, law enforcement referral |
| **Exploit** | Gather intelligence from the engagement | Collect new IOCs, TTPs, and intel from the incident |
| **Analyze** | Make sense of collected intel | Process and analyze the new intelligence |
| **Disseminate** | Share actionable intelligence | Distribute refined intel to stakeholders and partners |

**How F3EAD differs from the standard CTI lifecycle:**

| Standard Lifecycle | F3EAD |
|---|---|
| Planning-first | Action-first (Find the threat) |
| More analytical | More operational |
| Consumer-driven (PIRs) | Threat-driven (target-centric) |
| Suitable for all CTI | Best for threat hunting & IR |

**CTI application of F3EAD:**
- **Find:** Use threat intel, hunting, OSINT to identify active threat
- **Fix:** Monitor C2 traffic, track malware beaconing, maintain awareness of actor
- **Finish:** IR team responds, blocks, remediates
- **Exploit:** During/after IR, collect new malware samples, new IOCs, new TTPs
- **Analyze:** CTI team analyzes collected artifacts
- **Disseminate:** Updated intelligence shared with SOC, ISAC, partners

**F3EAD is cyclical:** Dissemination feeds back to Find for the next target or campaign.

---

### 3️⃣ Key Terms & Definitions

- **F3EAD:** Find, Fix, Finish, Exploit, Analyze, Disseminate
- **Target-centric:** Intelligence process focused on tracking specific actors/systems
- **Exploit (in F3EAD):** Extracting intelligence value from an engagement (NOT attacking back)
- **Fix:** Maintaining persistent awareness/surveillance of a target
- **Intelligence-led operations:** Using CTI to drive active security operations
- **Threat Hunting:** The "Find" phase in cybersecurity — proactively searching for hidden threats
- **Operationalization:** Converting intelligence into direct action (Find → Fix → Finish)

---

### 4️⃣ Examples

**Example — F3EAD in a ransomware incident:**
- **Find:** CTI analyst identifies ransomware group on dark web discussing targeting healthcare
- **Fix:** SOC monitors network logs for associated C2 patterns; sets up honeypots
- **Finish:** IR team detects and contains the intrusion; law enforcement notified
- **Exploit:** Malware sample recovered; new C2 domains extracted; actor's infrastructure mapped
- **Analyze:** CTI processes new IOCs; updates actor profile; maps new TTPs to ATT&CK
- **Disseminate:** Sends updated IOC bundle and flash report to FS-ISAC and internal SOC

---

### 5️⃣ Diagrams / Flow

```
F3EAD CYCLE:

        ┌─────────────────────────────────┐
        │                                 │
        ▼                                 │
     [FIND]                               │
  Identify threat/actor                   │
        │                                 │
        ▼                                 │
     [FIX]                                │
  Track and monitor                       │
        │                                 │
        ▼                                 │
    [FINISH]                              │
  Respond/Neutralize                      │
        │                                 │
        ▼                                 │
   [EXPLOIT]                              │
  Collect new intel                       │
        │                                 │
        ▼                                 │
   [ANALYZE]                              │
  Process & enrich                        │
        │                                 │
        ▼                                 │
 [DISSEMINATE] ──────────────────────────┘
  Share & loop back
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **F3EAD = Find, Fix, Finish, Exploit, Analyze, Disseminate** — memorize the full order
- "Exploit" in F3EAD means **intelligence exploitation** (collecting new intel from an engagement) — NOT hacking back
- F3EAD is more **operational** than the standard CTI lifecycle
- F3EAD originated in **military special operations targeting**
- F3EAD is particularly suited to **threat hunting and IR integration**

**Common traps:**
- ⚠️ "Finish" does NOT mean "attack the attacker" — it means contain/remediate/refer to law enforcement
- ⚠️ "Exploit" does NOT mean conducting offensive operations — it means extracting intelligence value
- ⚠️ F3EAD is **cyclical**, not linear

---

### 7️⃣ Quick Revision

- **F3EAD** = Find → Fix → Finish → Exploit → Analyze → Disseminate
- Originated in **military special operations** and adapted to CTI
- More **operational and action-oriented** than the standard 6-phase CTI lifecycle
- "Exploit" = collect intelligence FROM the incident (not attack back)
- Cyclical — Disseminate feeds back into Find for the next iteration

---

### 8️⃣ Practice Questions

**Q1.** In the F3EAD cycle, what does the "Exploit" phase mean in a CTI context?
- A) Launching a counter-attack against the threat actor
- B) Using a vulnerability against the attacker's system
- **C) Collecting and extracting intelligence value from an incident or engagement** ✅
- D) Identifying the threat actor's location

**Q2.** What is the correct order of the F3EAD cycle?
- A) Find, Analyze, Fix, Finish, Exploit, Disseminate
- **B) Find, Fix, Finish, Exploit, Analyze, Disseminate** ✅
- C) Find, Finish, Fix, Exploit, Analyze, Disseminate
- D) Fix, Find, Finish, Analyze, Exploit, Disseminate

**Q3.** The F3EAD model is best suited for which CTI activity?
- A) Setting quarterly PIRs
- B) Writing executive briefings
- **C) Intelligence-led threat hunting and incident response** ✅
- D) Compliance reporting

**Q4.** The "Fix" phase in F3EAD means:
- A) Remediate the system after an incident
- **B) Track and maintain awareness of the target threat actor or infrastructure** ✅
- C) Patch the vulnerable system
- D) Write the final intelligence report

**Q5.** How does F3EAD differ from the standard CTI lifecycle?
- A) F3EAD has fewer phases
- B) F3EAD is used only by government agencies
- **C) F3EAD is more operationally and action-oriented; the standard lifecycle is more planning and consumer-driven** ✅
- D) F3EAD does not include dissemination

---

## 18. Writing a Mock Intel Report

### 1️⃣ Core Concept
A **professional threat intelligence report** is a structured document that communicates actionable intelligence to a specific audience. CTI analysts must be able to write reports at **three levels**: tactical (IOC-focused), operational (campaign/actor-focused), and strategic (trend/risk-focused). Each serves a different audience and purpose.

---

### 2️⃣ Deep Dive

**Three levels of CTI reports:**

### 1. Tactical Report
- **Audience:** SOC analysts, IR teams, threat hunters
- **Content:** IOCs (IPs, domains, hashes), YARA/Sigma rules, detection guidance
- **Length:** 1–3 pages + IOC appendix
- **Format:** Technical, structured, machine-readable IOCs
- **Timeliness:** High — often produced within hours of threat discovery

### 2. Operational Report
- **Audience:** CTI team, IR leads, security managers
- **Content:** Campaign analysis, actor TTPs, kill chain mapping, infection chain
- **Length:** 3–10 pages
- **Format:** Narrative + ATT&CK mapping + IOCs
- **Timeliness:** Days to weeks

### 3. Strategic Report
- **Audience:** CISO, C-suite, Board, executives
- **Content:** Threat landscape trends, actor motivations, sector risk, business impact
- **Length:** 2–5 pages (no IOC dumps)
- **Format:** Business language, minimal technical jargon
- **Timeliness:** Monthly/quarterly

---

**Standard Intelligence Report Structure:**

```
1. Classification / TLP Marking
2. Report Title & Date
3. Executive Summary (3–5 sentences)
4. Key Findings (3–5 bullets)
5. Threat Overview / Background
6. Technical Analysis (for tactical/operational)
7. MITRE ATT&CK Mapping (table of technique IDs)
8. Recommendations / Mitigations
9. Conclusion
10. IOC Appendix (hashes, IPs, domains)
11. References / Sources
```

**Writing best practices:**
- **Lead with impact** — what matters most goes first
- **Use confidence levels** — "High confidence," "Medium confidence," "Low confidence"
- **Cite sources** — include report references or TLP-appropriate attribution
- **Be precise** — "The actor used T1059.001 (PowerShell)" vs "The actor used scripts"
- **Separate fact from assessment** — clearly distinguish "observed" from "assessed" or "likely"
- **Use active voice** — "The actor deployed LockBit ransomware" not "LockBit was deployed"

**Confidence levels (common CTI standard):**
- **High:** Multiple independent sources confirm; strong technical evidence
- **Medium:** Single source or some corroborating evidence; plausible
- **Low:** Limited evidence; speculative; based on pattern/precedent

---

### 3️⃣ Key Terms & Definitions

- **TLP Marking:** Applied at top of every report to control distribution
- **Executive Summary:** First section — most important finding in 3–5 sentences
- **Key Findings:** Bullet-point highlights of critical intelligence
- **Confidence Level:** Indicator of how certain the analyst is about an assessment
- **IOC Appendix:** Machine-readable section with all indicators for technical teams
- **ATT&CK Mapping:** Table matching observed behaviors to ATT&CK technique IDs
- **Source Attribution:** Crediting the source of intelligence (with appropriate TLP)
- **Operational Security (OPSEC):** Protecting methods and sources when writing reports

---

### 4️⃣ Examples

**Example — Tactical Report opening:**
> **TLP:AMBER | Tactical Intelligence Report**
> **Subject:** LockBit 3.0 IOCs — Active Campaign Targeting Healthcare Sector
> **Date:** [Date]
> **Executive Summary:** LockBit 3.0 ransomware is actively targeting US healthcare organizations. Three confirmed incidents in the past 72 hours. Initial access via RDP brute force. Immediate action: block listed IOCs and review RDP exposure.

**Example — Strategic Report opening:**
> **TLP:GREEN | Strategic Threat Landscape Brief**
> **Subject:** Q2 Ransomware Threat Landscape for Financial Services
> **Executive Summary:** Ransomware targeting financial institutions increased 34% in Q2. LockBit and BlackCat remain dominant. Organizations without MFA are 3x more likely to be victims. Board-level investment in identity security is recommended.

---

### 5️⃣ Diagrams / Flow

```
INTEL REPORT WRITING PROCESS:

[Intelligence Question / PIR / RFI]
        │
        ▼
[Collect Evidence / IOCs / TTPs]
        │
        ▼
[Analyze and Assess]
        │ Apply confidence levels
        ▼
[Draft Report Structure]
        │ TLP → Summary → Findings → Analysis → Recs → IOCs
        ▼
[Peer Review]
        │ Check: accuracy, bias, jargon, actionability
        ▼
[Classify + Distribute]
        │
        ▼
[Collect Feedback] → Improve next report
```

---

### 6️⃣ Exam Focus

**Most likely to be tested:**
- **Three report types**: Tactical (IOCs), Operational (campaigns), Strategic (trends)
- **Confidence levels** (High/Medium/Low) and when to use them
- TLP marking goes at the **top of every report**
- Executive Summary = **3–5 sentences max, lead with impact**
- **Separating fact from assessment** is a core professional standard

**Common traps:**
- ⚠️ IOC dumps are NOT executive reports — they're tactical appendices
- ⚠️ A report without recommendations is incomplete
- ⚠️ Forgetting TLP marking = poor OPSEC and governance violation

---

### 7️⃣ Quick Revision

- Three report levels: **Tactical** (IOCs/SOC), **Operational** (campaigns/IR), **Strategic** (trends/executives)
- Always start with **TLP marking** and **Executive Summary**
- Apply **confidence levels** (High/Medium/Low) to assessments
- **Separate facts** (observed) from **assessments** (analyzed/likely)
- Recommendations are **mandatory** — intel without guidance is incomplete

---

### 8️⃣ Practice Questions

**Q1.** A CTI analyst produces a report containing a list of malicious IPs, YARA rules, and detection guidance. This is a:
- **A) Tactical intelligence report** ✅
- B) Strategic intelligence report
- C) Operational threat profile
- D) Executive brief

**Q2.** Which element must appear at the TOP of every professional threat intelligence report?
- A) Executive Summary
- **B) TLP Classification** ✅
- C) IOC Appendix
- D) ATT&CK Mapping

**Q3.** An analyst states: "Based on C2 overlap and tooling similarities, we assess with medium confidence that APT28 is responsible." What does "medium confidence" indicate?
- A) The analyst is unsure if the event happened
- **B) The assessment is supported by some evidence but not fully corroborated** ✅
- C) The attribution is confirmed by multiple independent sources
- D) The IOCs were automatically generated

**Q4.** What is the primary difference between an operational and a strategic CTI report?
- A) Operational reports are longer
- B) Strategic reports include more IOCs
- **C) Operational reports focus on campaign TTPs; strategic reports focus on trends and business risk for executives** ✅
- D) Strategic reports are classified; operational are not

**Q5.** Which statement demonstrates correct analytical writing in a CTI report?
- A) "The attacker is APT28."
- **B) "We assess with high confidence, based on infrastructure and tooling overlap, that this activity is attributable to APT28."** ✅
- C) "Someone probably did this."
- D) "The IOCs suggest Russian activity (probably)."

---

# PHASE 4 — EXAM PREP

---

## 19. Practice Questions Master Set

### CTIGA-Style Full Practice Quiz

**SECTION A — Core Frameworks**

**Q1.** Which CTI lifecycle phase produces "finished intelligence"?
- A) Collection
- B) Processing
- **C) Analysis** ✅
- D) Planning

**Q2.** The Diamond Model's "technology axis" connects which two features?
- A) Adversary and Victim
- **B) Capability and Infrastructure** ✅
- C) Adversary and Capability
- D) Victim and Infrastructure

**Q3.** In the Cyber Kill Chain, what is the correct stage AFTER Installation?
- A) Delivery
- B) Exploitation
- **C) Command & Control** ✅
- D) Actions on Objectives

**Q4.** STIX 2.1 uses which data format?
- **A) JSON** ✅
- B) XML
- C) YAML
- D) CSV

**Q5.** TLP:GREEN allows sharing with:
- A) Named individuals only
- B) The entire public
- **C) A defined community or sector** ✅
- D) Only the receiving organization

---

**SECTION B — Core Domains**

**Q6.** Which type of CTI is best suited for a CISO making a budget decision about security investments?
- **A) Strategic** ✅
- B) Tactical
- C) Technical
- D) Operational

**Q7.** A CTI team has no defined PIRs and collects everything. What is the main risk?
- A) Too many IOCs to block
- B) Vendor lock-in
- **C) Undirected collection wastes resources and produces irrelevant intel** ✅
- D) Breach of GDPR automatically

**Q8.** Which framework most specifically guides CTI sharing in US government contexts?
- A) ISO 27001
- **B) NIST SP 800-150** ✅
- C) GDPR
- D) DORA

**Q9.** A CTI program uses a TIP, has defined PIRs, and shares via STIX/TAXII. What maturity level does this most likely represent?
- A) Level 1
- B) Level 2
- **C) Level 3** ✅
- D) Level 5

**Q10.** Which phase of Incident Response most benefits from CTI enrichment providing actor attribution and predicted next steps?
- A) Preparation
- **B) Detection and Analysis** ✅
- C) Recovery
- D) Containment

---

**SECTION C — Applied & Advanced**

**Q11.** In the F3EAD cycle, intelligence exploitation of a captured malware sample occurs at which phase?
- A) Find
- B) Fix
- C) Finish
- **D) Exploit** ✅

**Q12.** Analysis of Competing Hypotheses (ACH) helps analysts primarily by:
- A) Generating more IOCs
- **B) Reducing cognitive bias by systematically testing multiple hypotheses** ✅
- C) Automating threat detection
- D) Mapping TTPs to ATT&CK

**Q13.** An analyst uses MISP to automatically correlate the same C2 IP appearing in reports from multiple partner organizations. What MISP capability is being used?
- A) ATT&CK Navigator
- B) Galaxy integration
- **C) Automatic correlation** ✅
- D) TAXII push

**Q14.** When reading an APT report, which section should be checked FIRST to determine organizational relevance?
- A) IOC Appendix
- **B) Executive Summary** ✅
- C) Malware Technical Analysis
- D) YARA Rules

**Q15.** A CTI report states: "We assess with LOW confidence that FIN7 may be responsible." What does this mean?
- A) The attribution is confirmed
- B) There is no evidence at all
- **C) Evidence is limited or speculative; further investigation is needed** ✅
- D) The report should be marked TLP:RED

---

**SECTION D — Scenario-Based**

**Q16.** An IR team discovers malware beaconing to an unknown IP. They submit an urgent question to the CTI team. This is a:
- A) PIR
- **B) RFI** ✅
- C) STIX object
- D) TLP request

**Q17.** A CTI analyst discovers a report marked TLP:AMBER from an ISAC partner. They want to share it with their MSSP. Should they?
- A) Yes — MSSPs are trusted partners
- B) Yes — TLP:AMBER has no restrictions
- **C) No — TLP:AMBER limits sharing to the organization and clients on need-to-know only** ✅
- D) Only if converted to TLP:GREEN first

**Q18.** A national CERT tracks an APT actor's tools, infrastructure, and campaigns. They assign them the code "G0128." What type of ATT&CK object is this?
- A) Software (S-ID)
- B) Technique (T-ID)
- **C) Group (G-ID)** ✅
- D) Campaign (C-ID)

**Q19.** A company wants to move from CTI maturity Level 2 to Level 3. What is the MOST impactful first step?
- A) Hire 10 new CTI analysts
- **B) Implement a Threat Intelligence Platform and define formal PIRs** ✅
- C) Subscribe to all available threat feeds
- D) Achieve ISO 27001 certification

**Q20.** An executive report should ALWAYS end with:
- A) A full IOC appendix
- B) A technical malware analysis
- **C) Specific, prioritized recommendations for decision-makers** ✅
- D) Raw STIX bundle data

---

## 20. Framework Review & Comparison Table

### 1️⃣ Quick Comparison: All Major CTIGA Frameworks

| Framework | Created By | Purpose | Key Components | Best Used For |
|---|---|---|---|---|
| **CTI Lifecycle** | Industry standard | Structure the intel process | 6 phases: Plan→Collect→Process→Analyze→Disseminate→Feedback | All CTI operations |
| **MITRE ATT&CK** | MITRE | Document adversary behaviors | Tactics, Techniques, Sub-techniques, Procedures | Threat profiling, detection gap analysis |
| **Diamond Model** | Caltagirone et al. | Intrusion analysis | Adversary, Capability, Infrastructure, Victim | Pivoting, campaign analysis |
| **Cyber Kill Chain** | Lockheed Martin | Attack stage modeling | 7 stages: Recon→Weaponize→Deliver→Exploit→Install→C2→Actions | APT defense, defender actions |
| **STIX 2.1** | OASIS/CISA | CTI format/language | SDOs, SROs, Bundles | Structuring and sharing intel |
| **TAXII 2.1** | OASIS/CISA | CTI transport | Collections, Channels, HTTPS | Automated intel sharing |
| **TLP** | FIRST | Sharing control | RED, AMBER, AMBER+STRICT, GREEN, CLEAR | All intel sharing decisions |
| **F3EAD** | Military/adapted | Operational CTI cycle | Find, Fix, Finish, Exploit, Analyze, Disseminate | Threat hunting, IR operations |
| **NIST CSF** | NIST | Cybersecurity framework | Identify, Protect, Detect, Respond, Recover | Compliance, risk management |
| **ACH** | Heuer/SANS | Analytical technique | Hypothesis matrix | Bias reduction, attribution |

---

### 2️⃣ TLP Quick Reference Card

| Color | Sharing Allowed | Mnemonic |
|---|---|---|
| 🔴 RED | Named recipients ONLY | **R**estricted to **R**ecipients |
| 🟠 AMBER+STRICT | Organization ONLY | **A**mb+Strict = **A**lone in org |
| 🟡 AMBER | Org + need-to-know clients | **A**mber = **A**uthorized org+clients |
| 🟢 GREEN | Community (sector/ISAC) | **G**reen = **G**roup/community |
| ⬜ CLEAR | Unrestricted public | **C**lear = **C**ommunity/everyone |

---

### 3️⃣ ATT&CK Tactics — Quick Memory Aid

```
R-R-I-E-P-P-D-C-D-L-C-C-E-I

1.  Reconnaissance
2.  Resource Development
3.  Initial Access
4.  Execution
5.  Persistence
6.  Privilege Escalation
7.  Defense Evasion
8.  Credential Access
9.  Discovery
10. Lateral Movement
11. Collection
12. Command & Control
13. Exfiltration
14. Impact
```

---

### 4️⃣ Kill Chain vs ATT&CK — Key Differences

| Kill Chain | MITRE ATT&CK |
|---|---|
| 7 linear stages | 14 tactics (non-linear) |
| Lockheed Martin | MITRE Corporation |
| Defender-focused | Both attacker and defender perspectives |
| High-level attack flow | Detailed technique catalog |
| Limited to APT/external | Enterprise, Mobile, ICS matrices |
| Static | Continuously updated |

---

### 5️⃣ Diamond Model vs Kill Chain — Key Differences

| Diamond Model | Kill Chain |
|---|---|
| Shows relationships | Shows stages |
| Answers WHO/WHAT/WHERE | Answers WHEN/HOW |
| Event-based analysis | Campaign-stage analysis |
| Pivoting and attribution | Defensive action at each stage |
| 4 features | 7 stages |

---

### 6️⃣ CTI Lifecycle vs F3EAD — Key Differences

| CTI Lifecycle | F3EAD |
|---|---|
| Planning-first | Action-first |
| Consumer-driven (PIRs) | Threat actor-driven |
| 6 phases | 6 phases |
| Analytical/process-focused | Operational/action-focused |
| All CTI types | Best for hunting and IR |

---

### 7️⃣ MISP vs OpenCTI — Quick Reference

| | MISP | OpenCTI |
|---|---|---|
| Creator | CIRCL | Filigran |
| Focus | IOC sharing | Analysis/visualization |
| Data model | Events + Attributes | STIX 2.1 entities |
| Best for | ISACs, rapid sharing | Enterprise CTI teams |
| Galaxies/Connectors | Galaxies | Connectors |

---

## 21. Cold Attempt Strategy & Gap Analysis

### 1️⃣ Core Concept
A **cold attempt** is taking the exam or a full practice exam **without additional preparation** to identify your real knowledge gaps. The result then drives a **targeted gap analysis** and a **focused revision plan** in the final days before the exam.

---

### 2️⃣ Gap Analysis Process

**Step 1: Take a cold attempt (mock exam)**
- Answer all questions without reviewing notes
- Note which topics you guessed on
- Mark questions you were uncertain about even if you got them right

**Step 2: Analyze results by topic**

| Topic | Score | Confidence | Action |
|---|---|---|---|
| CTI Lifecycle | 5/5 | High | Review once, move on |
| ATT&CK | 3/5 | Medium | Revisit sub-techniques, tactic order |
| Diamond Model | 2/5 | Low | Full re-study needed |
| TLP | 5/5 | High | Quick revision only |
| F3EAD | 1/5 | Very Low | Priority revision |

**Step 3: Build targeted revision plan**
- Group topics: Strong / Needs Work / Critical Gap
- Allocate time: More time to Critical Gaps
- Use Quick Revision bullets from notes
- Re-attempt practice questions for weak areas

**Step 4: Re-attempt**
- Retake mock exam after targeted revision
- Track improvement
- Repeat until consistently above 80%

---

### 3️⃣ Exam Day Tips

**Before the exam:**
- Review Quick Revision bullets for all 20 topics (1 hour)
- Review the Framework Comparison Table
- Review TLP colors and their rules
- Memorize the order: CTI lifecycle phases, Kill Chain stages, F3EAD phases, ATT&CK tactics
- Get 8 hours sleep

**During the exam:**
- Read questions **carefully** — watch for "NOT," "EXCEPT," "MOST likely"
- **Eliminate wrong answers** first — CTI exams often have 2 clearly wrong and 2 plausible answers
- **Trust your first instinct** — don't second-guess unless you find a clear reason
- **Flag uncertain questions** — return to them after completing others
- Translate scenarios into framework language — "who did what to whom" → Diamond Model

**Common exam traps to watch for:**
- ⚠️ "Which phase comes AFTER X?" — Know exact sequence of CTI lifecycle, Kill Chain, F3EAD
- ⚠️ STIX vs TAXII confusion — STIX=format, TAXII=transport
- ⚠️ TLP:GREEN ≠ public — it's community only
- ⚠️ "Exploit" in F3EAD ≠ offensive hacking — it means intel collection
- ⚠️ Operational vs Strategic CTI — know which audience uses each
- ⚠️ ATT&CK Tactics vs Techniques — Tactic=goal, Technique=method

---

### 4️⃣ Last-Minute Mnemonics

**CTI Lifecycle:** "**P**lease **C**ome **P**lay **A**ll **D**ay **F**riday"
→ Plan, Collect, Process, Analyze, Disseminate, Feedback

**Kill Chain:** "**R**eal **W**arriors **D**on't **E**at **I**nside **C**hina **A**lone"
→ Recon, Weaponize, Deliver, Exploit, Install, C2, Actions

**F3EAD:** "**F**ind **F**ish, **F**ry **E**verything, **A**nd **D**ish"
→ Find, Fix, Finish, Exploit, Analyze, Disseminate

**NIST CSF:** "**I** **P**refer **D**oing **R**eally **R**ewarding (work)"
→ Identify, Protect, Detect, Respond, Recover

**Diamond Model:** "**A**ll **C**ompetent **I**ntel **V**ets"
→ Adversary, Capability, Infrastructure, Victim

---

### 5️⃣ Final Confidence Check — Self-Assessment

Rate yourself (1–5) on each topic before exam day:

| Topic | Your Rating | Target |
|---|---|---|
| CTI Lifecycle | __ | 4+ |
| MITRE ATT&CK | __ | 4+ |
| Diamond Model | __ | 4+ |
| Cyber Kill Chain | __ | 5 |
| STIX / TAXII | __ | 4+ |
| TLP | __ | 5 |
| Intel Governance | __ | 4+ |
| CTI in SOC/IR | __ | 4+ |
| PIR / RFI | __ | 5 |
| Intel Maturity | __ | 4+ |
| Executive Reporting | __ | 4+ |
| Compliance | __ | 4+ |
| APT Reports | __ | 4+ |
| ATT&CK CTI Training | __ | 4+ |
| OpenCTI / MISP | __ | 4+ |
| SANS FOR578 / ACH | __ | 4+ |
| F3EAD | __ | 5 |
| Intel Report Writing | __ | 4+ |
| Practice Questions | __ | 4+ |
| Framework Review | __ | 5 |

Any topic rated **3 or below** = needs focused review before exam day.

---

# 🏁 END OF CTIGA EXAM NOTES

> **Total Topics Covered:** 21 across 4 Phases
> **Total Practice Questions:** 80+ MCQs
> **Study Path:** Phase 1 (Week 1) → Phase 2 (Week 2) → Phase 3 (Week 3) → Phase 4 (Week 4)
> **Good luck on your CTIGA exam! 🛡️**
