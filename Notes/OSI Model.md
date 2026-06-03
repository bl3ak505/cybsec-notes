### 📌 OSI Model — Full Deep Dive

**Definition**

- 7-layer reference model that defines how data travels from one device to another across a network. Each layer has one job and does it well.

---
## Quick Cheat Sheet 🧠

|Layer|Name|Device|PDU|Attack Example|
|---|---|---|---|---|
|7|Application|Proxy, Firewall|Data|SQLi, XSS, RCE|
|6|Presentation|—|Data|SSL stripping|
|5|Session|—|Data|Session hijacking|
|4|Transport|Firewall|Segment|SYN flood, port scan|
|3|Network|Router|Packet|IP spoofing, ICMP tunnel|
|2|Data Link|Switch|Frame|ARP poison, MAC flood|
|1|Physical|Hub, NIC|Bits|Sniffing, jamming|


---

## Layer 7 — Application Layer

**What it is:** The layer closest to the user. This is where actual applications interact with the network. It doesn't mean the app itself (like Chrome) — it means the **protocols** that apps use.

**Job:** Provide network services directly to end-user applications.

**Protocols:**

|Protocol|Port|Purpose|
|---|---|---|
|HTTP|80|Web browsing|
|HTTPS|443|Encrypted web|
|FTP|20/21|File transfer|
|SSH|22|Secure shell|
|DNS|53|Domain → IP resolution|
|SMTP|25|Sending email|
|IMAP/POP3|143/110|Receiving email|
|DHCP|67/68|Auto IP assignment|
|SNMP|161|Network device management|

**How it works:**

- When you search something, your browser constructs an HTTP GET request at this layer
- DNS query happens here to resolve the domain name
- The request gets handed down to Layer 6

**Hacker relevance 🎯**

- SQLi, XSS, SSRF, IDOR, RCE — all L7 attacks
- Gobuster, Nikto, Burp Suite all operate here
- WAFs (Web App Firewalls) defend at this layer

---

## Layer 6 — Presentation Layer

**What it is:** The translator of the OSI model. Converts data into a format the application can understand, and handles encryption/decryption + compression.

**Job:** Data formatting, encoding, encryption, compression.

**Key responsibilities:**

- **Encoding:** Converts data to a common format (ASCII, Unicode, EBCDIC)
- **Encryption/Decryption:** TLS/SSL operates here — encrypts data before sending, decrypts on arrival
- **Compression:** Reduces data size (JPEG, MP3, ZIP)

**Formats & standards:**

- JPEG, PNG, GIF (images)
- MPEG, MP4 (video)
- ASCII, UTF-8 (text encoding)
- SSL/TLS (encryption)

**How it works:**

- Sender: takes raw app data → encodes it → compresses it → encrypts it → passes to L5
- Receiver: decrypts → decompresses → decodes → passes to L7

**Hacker relevance 🎯**

- SSL stripping attacks target this layer
- Weak cipher exploitation (old TLS 1.0, RC4, etc.)
- Encoding tricks in payloads — Base64, URL encoding, Unicode bypass = abusing L6 behavior

---

## Layer 5 — Session Layer

**What it is:** Manages the **conversation** between two devices. It sets up, maintains, and tears down sessions.

**Job:** Session establishment, maintenance, synchronization, termination.

**Key responsibilities:**

- **Authentication** — verifies who you're talking to before a session starts
- **Authorization** — confirms what you're allowed to do
- **Session management** — keeps track of which data belongs to which session
- **Checkpointing** — if a large file transfer fails midway, L5 can resume from a checkpoint instead of restarting

**Protocols:**

- NetBIOS — session management on Windows networks
- RPC (Remote Procedure Call) — used heavily in Windows/AD environments
- PPTP — VPN tunneling
- SMB (partly) — file sharing on Windows

**How it works:**

- Three phases: **Establishment → Data Transfer → Termination**
- Assigns a **session ID** to track the conversation
- Keeps multiple sessions separate (that's why you can have multiple browser tabs open simultaneously)

**Hacker relevance 🎯**

- Session hijacking lives here — steal the session ID, steal the session
- Pass-the-Hash, Pass-the-Ticket attacks abuse session authentication (Kerberos, NTLM)
- SMB exploitation (EternalBlue) involves this layer

---

## Layer 4 — Transport Layer

**What it is:** The most technically important layer for networking. Handles **end-to-end communication**, breaks data into segments, manages flow control and error recovery.

**Job:** Reliable (or unreliable) data delivery between endpoints using port numbers.

**Two main protocols:**

||TCP|UDP|
|---|---|---|
|Connection|Connection-oriented (3-way handshake)|Connectionless|
|Reliability|Guaranteed delivery, retransmits lost packets|Fire and forget|
|Order|Packets arrive in order|No guaranteed order|
|Speed|Slower|Faster|
|Use case|HTTP, SSH, FTP, SMTP|DNS, VoIP, gaming, streaming|

**TCP 3-Way Handshake:**

```
Client → SYN       → Server
Client ← SYN-ACK   ← Server
Client → ACK        → Server
[Connection established]
```

**TCP 4-Way Termination:**

```
Client → FIN
Server → ACK
Server → FIN
Client → ACK
```

**Port ranges:**

- 0–1023 → Well-known ports (HTTP, SSH, FTP)
- 1024–49151 → Registered ports
- 49152–65535 → Dynamic/ephemeral ports

**Flow control:** TCP uses a **sliding window** — controls how much data is sent before requiring an ACK, prevents overwhelming the receiver.

**Error control:** Sequence numbers track every byte — if a packet is lost, TCP retransmits it.

**Hacker relevance 🎯**

- **SYN flood** — send thousands of SYN packets, never complete handshake → server holds half-open connections → DoS
- **Port scanning** — nmap's SYN scan (`-sS`) sends SYN, reads SYN-ACK vs RST to detect open ports
- **TCP session hijacking** — predict sequence numbers, inject packets mid-session
- **UDP attacks** — UDP flood, DNS amplification all abuse connectionless nature

---

## Layer 3 — Network Layer

**What it is:** Responsible for **logical addressing and routing** — getting packets from source to destination across multiple networks.

**Job:** IP addressing, routing, path determination, packet forwarding.

**Key protocols:**

|Protocol|Purpose|
|---|---|
|IP (v4/v6)|Logical addressing and packet delivery|
|ICMP|Error messages and diagnostics (ping lives here)|
|ARP|Resolves IP → MAC (bridges L3 and L2)|
|OSPF, BGP, RIP|Routing protocols — how routers share path info|
|IPSec|Network-layer encryption|

**How IP addressing works:**

- Every device gets an IP address (logical, can change)
- IPv4: 32-bit → `192.168.1.1`
- IPv6: 128-bit → `2001:0db8::1`
- Subnet mask determines network vs host portion
- Routers use **routing tables** to decide where to forward each packet

**How routing works:**

- Packet arrives at router → router checks destination IP → looks up routing table → forwards to next hop
- This repeats hop-by-hop until packet reaches destination
- TTL (Time To Live) decrements at each hop — prevents infinite loops

**Hacker relevance 🎯**

- **IP spoofing** — forge source IP in packet headers
- **ICMP attacks** — ping of death, ICMP tunneling (data exfil through ping)
- **ARP poisoning** — send fake ARP replies to link your MAC with someone else's IP → MITM
- **Traceroute** — maps network path using TTL manipulation
- `nmap -sn` ping sweep operates at this layer
[[Nmap syntaxes]]
[[Network Scanning]]
---

## Layer 2 — Data Link Layer

**What it is:** Handles communication between devices on the **same local network** using physical (MAC) addresses. Ensures error-free transmission over the physical link.

**Job:** Physical addressing (MAC), framing, error detection, access control.

**Two sublayers:**

- **LLC (Logical Link Control)** — error checking, flow control
- **MAC (Media Access Control)** — hardware addressing, controls access to the physical medium

**Key concepts:**

|Concept|Detail|
|---|---|
|MAC Address|48-bit hardware address burned into NIC (e.g., `AA:BB:CC:DD:EE:FF`)|
|Frame|L2 data unit — wraps packet with MAC src/dst + error check|
|FCS|Frame Check Sequence — detects transmission errors (CRC)|
|VLAN|Virtual LAN — logical segmentation at L2|

**How it works:**

- Data from L3 (packet) gets wrapped in a **frame**
- Frame has: Destination MAC | Source MAC | EtherType | Payload | FCS
- Switch reads destination MAC, looks up MAC table, forwards only to correct port
- If MAC unknown, switch **floods** the frame to all ports (that's how ARP discovery works)

**Protocols:** Ethernet (802.3), Wi-Fi (802.11), PPP, HDLC, STP (Spanning Tree)

**Hacker relevance 🎯**

- **ARP spoofing/poisoning** — send fake ARP, redirect traffic through you → MITM (arpspoof, Ettercap)
- **MAC flooding** — overflow switch's MAC table → switch goes into hub mode → you see all traffic
- **VLAN hopping** — abuse trunk ports to jump between VLANs you shouldn't access
- **802.1X bypass** — bypass network access control at L2
- Wireshark captures frames at this layer

---

## Layer 1 — Physical Layer

**What it is:** The actual physical transmission of raw **bits** (1s and 0s) over a medium. No addressing, no logic — just signal.

**Job:** Transmit raw bitstream over physical medium (electrical, optical, or radio signals).

**Transmission media:**

|Medium|Type|Example|
|---|---|---|
|Copper cable|Electrical signals|Ethernet (Cat5e, Cat6)|
|Fiber optic|Light pulses|Backbone networks, ISPs|
|Wireless|Radio waves|Wi-Fi, Bluetooth, Cellular|
|Coaxial|Electrical|Cable TV/internet|

**Key concepts:**

- **Bandwidth** — max data rate of the medium
- **Throughput** — actual data rate achieved
- **Latency** — delay in signal travel
- **Attenuation** — signal weakens over distance
- **Noise** — interference that corrupts signal
- **Duplex** — half duplex = one direction at a time; full duplex = both simultaneously

**Devices at this layer:** Hubs, repeaters, cables, network cards (NIC), modems

**Encoding schemes:**

- NRZ, Manchester encoding — how 1s and 0s are represented as voltage changes
- Wi-Fi uses OFDM (Orthogonal Frequency Division Multiplexing)

**Hacker relevance 🎯**

- **Packet sniffing in promiscuous mode** — NIC captures ALL frames, not just its own (Wireshark, tcpdump)
- **Evil twin attack** — broadcast stronger Wi-Fi signal to hijack connections (hostapd-wpe)
- **Jamming** — flood radio frequencies to DoS wireless networks
- **Physical access attacks** — plug into an exposed port = instant L1 access, game over

---

## Full Encapsulation Flow (Sending Data)

```
Application (L7)   → [Data]
Presentation (L6)  → [Encrypted Data]
Session (L5)       → [Session Header | Encrypted Data]
Transport (L4)     → [TCP Header | Data]  ← Segment
Network (L3)       → [IP Header | TCP Header | Data]  ← Packet
Data Link (L2)     → [MAC Header | IP Header | TCP Header | Data | FCS]  ← Frame
Physical (L1)      → 010110100110101010...  ← Bits
```

Receiving side strips each header going back up 🔼

---

