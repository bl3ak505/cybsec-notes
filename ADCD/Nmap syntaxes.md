# Nmap: The Breakdown

## Basic Syntax

```
nmap [scan type] [options] [target]
```

Target can be IP, range (`192.168.1.1-254`), CIDR (`192.168.1.0/24`), or hostname.

---

## Host Discovery

```bash
nmap -sn 192.168.1.0/24          # Ping sweep, no port scan
nmap -Pn 192.168.1.1             # Skip host discovery (treat as up)
nmap -PS 192.168.1.1             # TCP SYN ping
nmap -PA 192.168.1.1             # TCP ACK ping
nmap -PU 192.168.1.1             # UDP ping
```

---

## Scan Types

```bash
nmap -sS 192.168.1.1             # SYN scan (stealth, default w/ root)
nmap -sT 192.168.1.1             # TCP connect (no root needed)
nmap -sU 192.168.1.1             # UDP scan (slow but crucial)
nmap -sA 192.168.1.1             # ACK scan (firewall mapping)
nmap -sN 192.168.1.1             # NULL scan
nmap -sF 192.168.1.1             # FIN scan
nmap -sX 192.168.1.1             # Xmas scan
```

---

## Port Specification

```bash
nmap -p 80 192.168.1.1           # Single port
nmap -p 80,443,8080 192.168.1.1  # Multiple ports
nmap -p 1-1000 192.168.1.1       # Port range
nmap -p- 192.168.1.1             # ALL 65535 ports
nmap --top-ports 100 192.168.1.1 # Top 100 common ports
```

---

## Service & OS Detection

```bash
nmap -sV 192.168.1.1             # Service/version detection
nmap -sV --version-intensity 9   # Max version intensity
nmap -O 192.168.1.1              # OS detection (needs root)
nmap -A 192.168.1.1              # Aggressive: OS + version + scripts + traceroute
```

---

## NSE Scripts

```bash
nmap -sC 192.168.1.1                        # Default scripts
nmap --script=vuln 192.168.1.1              # Vuln scan
nmap --script=http-enum 192.168.1.1         # Web enumeration
nmap --script=smb-vuln* 192.168.1.1         # All SMB vuln scripts
nmap --script=banner 192.168.1.1            # Banner grabbing
nmap --script=ftp-anon 192.168.1.1 -p 21   # Anonymous FTP check
nmap --script=ssh-brute 192.168.1.1 -p 22  # SSH brute force
```

Scripts live in `/usr/share/nmap/scripts/` — browse them.

---

## Timing & Performance

```bash
nmap -T0   # Paranoid (IDS evasion, painfully slow)
nmap -T1   # Sneaky
nmap -T2   # Polite
nmap -T3   # Normal (default)
nmap -T4   # Aggressive (fast, reliable networks)
nmap -T5   # Insane (fast + noisy, you don't care about stealth)
```

---

## Evasion & Spoofing

```bash
nmap -f 192.168.1.1                        # Fragment packets
nmap -D RND:10 192.168.1.1                 # Decoy scan (10 random decoys)
nmap -D 10.0.0.1,10.0.0.2,ME 192.168.1.1  # Specific decoys
nmap -S 10.0.0.5 192.168.1.1              # Spoof source IP
nmap --source-port 53 192.168.1.1          # Spoof source port (DNS bypass trick)
nmap --data-length 25 192.168.1.1          # Append random data to packets
nmap --randomize-hosts 192.168.1.0/24      # Scan hosts in random order
```

---

## Output Formats

```bash
nmap -oN output.txt    # Normal text
nmap -oX output.xml    # XML
nmap -oG output.gnmap  # Grepable
nmap -oA output        # All three at once (recommended always)
```

---

## My Go-To Combos

**Quick recon:**

```bash
nmap -sS -T4 --top-ports 1000 -oA quick_scan 192.168.1.1
```

**Full aggressive:**

```bash
nmap -A -T4 -p- -oA full_scan 192.168.1.1
```

**Stealth + evasion:**

```bash
nmap -sS -T2 -f -D RND:5 --source-port 53 -oA stealth 192.168.1.1
```

**Vuln sweep:**

```bash
nmap -sV --script=vuln -T4 -oA vulns 192.168.1.1
```

**UDP top ports (don't skip this, people always skip this):**

```bash
nmap -sU --top-ports 200 -T4 192.168.1.1
```

---

UDP scans and `-p-` full scans are where most people get lazy. Don't be most people.