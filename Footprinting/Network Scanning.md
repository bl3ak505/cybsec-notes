# 📌 Network Scanning

> Actively discovering hosts, open ports, running services, and OS information across a target's IP space.

---

## 🔍 Definition

**Network Scanning** is an active reconnaissance technique that sends packets to a target IP range and analyzes responses to discover live hosts, open ports, running services, and operating systems. It's the bridge between passive footprinting and actual exploitation.

---

## 🧠 Core Concepts

- **Host discovery** — determining which IPs are alive (ping sweep, ARP scan)
- **Port scanning** — identifying which TCP/UDP ports are open
- **Service detection** — identifying what software is running on each open port
- **OS fingerprinting** — guessing the OS from TCP/IP stack behavior
- **Stealth vs noise** — different scan types generate different amounts of logs/alerts

---

## ⚙️ Scan Types

| Scan Type | Nmap Flag | How It Works | Stealth |
|-----------|-----------|-------------|---------|
| **TCP Connect** | `-sT` | Full 3-way handshake, logged | Low |
| **SYN Stealth** | `-sS` | Sends SYN, doesn't complete handshake | Medium |
| **UDP Scan** | `-sU` | Sends UDP packets, slow | Medium |
| **FIN Scan** | `-sF` | Sends FIN — closed ports respond with RST | High |
| **NULL Scan** | `-sN` | No flags set — closed ports respond with RST | High |
| **XMAS Scan** | `-sX` | FIN+PSH+URG flags — closed ports RST | High |
| **ACK Scan** | `-sA` | Tests firewall rules, not open ports | High |
| **Ping Sweep** | `-sn` | Host discovery only, no port scan | Medium |

---

## 🛠️ Tools & Commands

### Nmap (The Standard)

```bash
# Basic host discovery
nmap -sn 192.168.1.0/24                    # ping sweep, no port scan
nmap -sn -PE -PP -PM 192.168.1.0/24       # ICMP echo, timestamp, netmask

# Port scanning
nmap -sS target.com                         # SYN scan (default, requires root)
nmap -sT target.com                         # TCP connect (no root needed)
nmap -p 1-65535 target.com                  # all ports
nmap --top-ports 1000 target.com            # top 1000 common ports
nmap -p 22,80,443,8080,8443 target.com      # specific ports

# Service + version detection
nmap -sV target.com                         # version detection
nmap -sV --version-intensity 9 target.com   # aggressive version detection
nmap -O target.com                          # OS detection
nmap -A target.com                          # everything: OS, version, scripts, traceroute

# NSE scripts
nmap -sC target.com                         # default scripts
nmap --script vuln target.com               # vulnerability scripts
nmap --script http-headers target.com       # HTTP headers
nmap --script=banner target.com             # service banners

# Output
nmap -oN output.txt target.com              # normal text
nmap -oX output.xml target.com              # XML (for parsing)
nmap -oG output.gnmap target.com            # grepable
nmap -oA output target.com                  # all three formats
```

### RustScan (Fast + Nmap Pipe)

```bash
# Find ports fast, then Nmap for details
rustscan -a target.com -- -sV -sC
rustscan -a target.com -p 1-65535 -- -A
rustscan -b 500 -a 192.168.1.0/24          # batch size 500 for range scan
```

### Masscan (Internet-Scale Speed)

```bash
# Extremely fast, full port range
masscan -p1-65535 target.com --rate=1000
masscan -p80,443,8080,8443 192.168.1.0/24 --rate=5000
masscan -p1-65535 --rate=10000 -oG masscan.txt target.com
```

### Naabu (Bug Bounty Focused)

```bash
# Fast, reliable, Go-based
naabu -host target.com -top-ports 1000
naabu -list subs.txt -p 80,443,8080,8443 -silent
naabu -host target.com -p - -silent        # all ports
```

### smap (Passive — Shodan Data)

```bash
# Zero packets sent to target
smap target.com
smap -iL targets.txt -oX output.xml
```

---

## 📖 Key Terminology

| Term | Meaning |
|------|---------|
| **Open port** | Port actively accepting connections |
| **Closed port** | Port reachable but no service listening — responds with RST |
| **Filtered port** | Firewall dropping packets — no response |
| **SYN scan** | Sends SYN, gets SYN/ACK (open) or RST (closed), never completes handshake |
| **TTL** | Time To Live — used in OS fingerprinting (Linux=64, Windows=128, Cisco=255) |
| **Ping sweep** | ICMP echo requests across IP range to find live hosts |
| **NSE** | Nmap Scripting Engine — Lua scripts for service-specific enumeration |
| **Decoy scan** | `-D` flag — spoofs source IPs to confuse IDS |

---

## 🔥 Important Details & Gotchas

- **SYN scan requires root** — use `sudo nmap -sS` or run as root
- **UDP scanning is slow** — UDP has no handshake, Nmap waits for ICMP unreachable to confirm closed. Use `-p` to target specific UDP ports
- **Firewalls return filtered, not closed** — filtered = packets dropped, not rejected. Can be stateful firewall
- **ICMP may be blocked** — if `-sn` shows no hosts, try `-Pn` to skip ping and scan anyway
- **`-Pn`** — skips host discovery, scans all IPs as if alive. Use when ICMP is blocked
- **Masscan vs Nmap** — Masscan is for speed (full range), Nmap is for accuracy (service details). Use both
- **`-T4` for faster scans** — timing template, T0=paranoid, T5=insane. T4 is good balance for labs
- **Rate limiting with IDS** — `-T1` or `-T2` with random delay for evasion: `--scan-delay 500ms`

---

## 🌍 Standard Scan Workflow

```bash
# 1. Fast all-port discovery
rustscan -a target.com -p 1-65535 -- -Pn | tee ports.txt

# 2. Detailed service detection on found ports
nmap -sV -sC -p $(cat ports.txt | grep open | cut -d/ -f1 | tr '\n' ',') target.com

# 3. Vuln scripts on interesting ports
nmap --script vuln -p 80,443,22,21 target.com

# 4. UDP top ports
nmap -sU --top-ports 100 target.com
```

---

## 🔗 Related Topics

- [[Footprinting Tools — Complete Reference]]
- [[Banner Grabbing]]
- [[DNS Enumeration]]
- [[Bug Bounty Recon Pipeline]]
- [[Nmap syntaxes]]
- [[CEH_v13__Module_02_Footprinting_and_Reconnaissance.pdf#page=24]]

---

## 📚 Sources / Further Reading

- Nmap official docs — nmap.org/book/
- NSE script library — nmap.org/nsedoc/
- RustScan GitHub — RustScan/RustScan
- Masscan GitHub — robertdavidgraham/masscan
