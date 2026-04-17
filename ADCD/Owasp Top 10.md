

# All 10 Simply Explained

**A01 — Broken Access Control** 👑 still #1 [[Broken Access Control]]

> Users accessing things they shouldn't. SSRF now lives here too.

**A02 — Security Misconfiguration** 📈 big jump [[Security Misconfiguration]]

> Wrong settings everywhere. 100% of tested apps had this. Literally everyone.

**A03 — Software Supply Chain Failures** 🆕 expanded [[Software Supply Chain Failures]]

> Your code is fine but the libraries your code uses got compromised. SolarWinds, Log4Shell energy.

**A04 — Cryptographic Failures** 📉  [[Cryptographic Failures]]

> Bad or missing encryption. Passwords stored in MD5, HTTP instead of HTTPS.

**A05 — Injection** 📉 fell from #3 [[Injection]]

> SQLi, command injection, XSS. The OG. Still deadly, just more understood now.

**A06 — Insecure Design** 📉 [[Insecure Design]]

> Bad architecture decisions baked in from the start. No threat modeling.

**A07 — Auth Failures** ➡️ same [[Authentication Failures]]

> Weak passwords, no MFA, broken session management.

**A08 — Software & Data Integrity Failures** ➡️ same [[Software & Data Integrity Failures]]

> Trusting code/data without verifying it. Unsigned updates, insecure deserialization.

**A09 — Logging & Alerting Failures** ➡️ same [[Security Logging & Monitoring Failures]]

> Attacks happening with nobody noticing. No logs, no alerts, no visibility.

**A10 — Mishandling of Exceptional Conditions** 🆕 brand new [[Mishandling of Exceptional Conditions]]

> App crashes badly and leaks info, or fails open instead of failing safe. The "what happens when things go wrong" category.