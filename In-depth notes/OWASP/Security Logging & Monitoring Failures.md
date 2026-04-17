### 📌 A09: Security Logging & Monitoring Failures 

**Definition**

- Occurs when an application **fails to log security-relevant events, monitor them, or alert on suspicious activity** — giving attackers free reign to operate undetected for extended periods.

> you got breached 6 months ago and you still don't know about it. the attacker moved in, set up furniture, and started forwarding your mail 🏠💀

---

**Core Concepts**

- **Logging** = recording what happened (who, what, when, where)
- **Monitoring** = actively watching those logs for suspicious patterns
- **Alerting** = notifying someone when something bad is detected
- All three can fail independently — and often do
- This vuln doesn't _cause_ the initial breach — it **allows it to go undetected and uncontained**
- Average breach dwell time (attacker inside before detection): **200+ days** — largely because of this category
- Without logs: no forensics, no incident response, no evidence, no learning 😬

---

**How It Works**

**Attacker perspective — why this is a gift:**

- Brute-force 10,000 login attempts → no alert fires → credentials found → logged in
- Exploit SQLi → dump entire database → no anomaly detected in query logs
- Exfiltrate 50GB of data at 3am → no alert on unusual outbound traffic
- Escalate privileges → no log of the privilege change → moves laterally undetected
- Cover tracks → if logs exist but aren't protected → attacker deletes/tampers them → incident response is blind

**What typically goes wrong:**

- Login failures not logged → brute-force invisible
- Successful logins not logged → can't tell who accessed what
- Logs exist but nobody reads them → equivalent to not having them
- No SIEM/alerting → logs are a graveyard of unread text files
- Logs stored on same server as app → compromised app = compromised logs
- No log integrity protection → attacker edits logs post-compromise
- Sensitive data (passwords, PII) accidentally logged → logging creates a new vuln

---

**Key Terminology**

|Term|Meaning|
|---|---|
|SIEM|Security Information & Event Management — aggregates + correlates logs, triggers alerts (Splunk, ELK, QRadar)|
|Log aggregation|Centralizing logs from multiple sources into one place|
|Dwell time|Time attacker spends inside a network before detection|
|Audit trail|Chronological record of security-relevant events|
|Alerting threshold|Condition that triggers a security notification (e.g., 10 failed logins in 1 min)|
|SOC|Security Operations Center — team that monitors alerts 24/7|
|Log tampering|Attacker modifying/deleting logs to hide activity|
|WORM storage|Write Once Read Many — tamper-proof log storage|
|IOC|Indicator of Compromise — evidence that an attack occurred (suspicious IP, hash, pattern)|
|Log retention|How long logs are kept — too short = can't investigate old incidents|

---

**Important Technical Details**

- **What MUST be logged** (bare minimum):
    
    - All authentication events (success AND failure)
    - Access control failures (403s, privilege escalation attempts)
    - Input validation failures (SQLi attempts, XSS payloads hitting WAF)
    - Session management events (creation, invalidation, anomalies)
    - Admin/privileged actions
    - Application errors and exceptions
- **What must NOT be logged** (avoid creating new vulns):
    
    - Passwords (even failed ones)
    - Session tokens
    - Credit card numbers / PII
    - Any sensitive user data
- **Log format best practices:**
    
    - Timestamp (UTC, not local time — timezones cause forensic chaos)
    - Source IP
    - User/account involved
    - Action performed
    - Success/failure
    - Affected resource
- **SIEM tools to know:**
    
    - Splunk — industry standard, expensive
    - ELK Stack (Elasticsearch + Logstash + Kibana) — open source, widely used
    - QRadar — IBM's enterprise SIEM
    - Wazuh — open source, popular in labs
- **Log protection:**
    
    - Ship logs to **separate, hardened server** immediately — attacker shouldn't be able to reach them
    - WORM storage or immutable cloud logging (AWS CloudTrail, Azure Monitor)
    - Hash log files periodically → detect tampering
- **Detection rules example** (what a SIEM should alert on):
    
    - 5+ failed logins in 60 seconds from same IP → brute force
    - Login from new country immediately after domestic login → account takeover
    - Privilege escalation outside business hours → sus 👀
    - Large outbound data transfer at 3am → exfiltration

---

**Real-World Example**

- **Target breach (2013)**: Attackers were inside Target's network for **3 weeks** before anyone noticed — security monitoring tools actually fired alerts but the SOC team ignored them 🤦 40 million credit cards gone. The logs existed. Nobody acted.
- **Equifax (2017)**: Attackers had **79 days** of undetected access — expired SSL certificate meant encrypted traffic wasn't being inspected, so malicious exfiltration was invisible to monitoring tools
- **Every APT ever**: Nation-state attackers specifically target logging infrastructure first — disable or blind the logs before doing anything else. If your logs are on the same box as your app, you've already lost

---

> 🔑 **CEH angle:** Know SIEM tools by name (Splunk, QRadar, ELK). Understand that this category is about **detection & response** not prevention — it's the safety net when everything else fails. Log _what matters_, protect log _integrity_, _centralize_ storage, and actually _monitor_ — all four gaps are testable. Dwell time stat (200+ days average) comes up in exam contexts. Know that insufficient logging violates non-repudiation as a security principle. 🔒