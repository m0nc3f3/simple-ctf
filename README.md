---

title: "Simple CTF: Net Sec Challenge Write-up"  
author: moncef fennan  
date: 2025-11-03  
tags:

- ctf
    
- tryhackme
    
- nmap
    
- enumeration
    
- ssh
    
- web
    

---

# Simple CTF: Write-up

_Author: moncef fennan — Date: 2025-11-03_

## Table of Contents

- [🚀 Scanning & Reconnaissance]
    
- [🕵️ Service Enumeration & Flag Hunting]
    
    - [Which services under port 1000?]
        
    - [Higher port / SSH]
        
    - [Web app (CMS Made Simple) — CVE & vulnerability]
        
    - [Password & Cracking]
        
    - [SSH login & user flag]
        
    - [Other local users]
    - [Privilege escalation (sudo/vim)]
- [💻 Web Challenge (if any)]
- [🏁 Conclusion](https://chatgpt.com/c/6907e2b1-c354-832d-a5ce-8bc1eabe91df#-conclusion)
    

---

## 🚀 Scanning & Reconnaissance

Start with a full TCP port scan to discover services.

```bash
nmap -sT -p- <TARGET_IP>
```

<img src="/images/nmap-scan.png" alt="this is nmap scan">

> [!NOTE]  
> **Question:** How many services are running under port 1000?  
> **Answer:** `2`

---

## 🕵️ Service Enumeration & Flag Hunting

We go service by service: grab banners, poke the web app, and hunt for flags.

### Higher port / SSH

> [!NOTE]  
> **Question:** What is running on the higher port?  
> **Answer:** The service on port `2222` is **ssh**.

SSH on a non-standard port — nothing fancy, just remember to connect with `-p 2222`

---

### Web app (CMS Made Simple) — CVE & vulnerability

Directory enumeration showed a `/simple` path which pointed to **CMS Made Simple**. I checked Exploit‑DB and found an exploit for it.
<img src="/images/enumeration.png" alt="this is enumeration using gobuster">

<img src="directory simple.png" alt="this is simple directory">

`I confirmed the CVE and vulnerability type from the exploit details.`
<img src="/images/cms made simple cve.png" alt="this is the cve">

> [!NOTE]  
> **Question:** What's the CVE you're using against the application?  
> **Answer:** `CVE-2019-9053`

> [!NOTE]  
> **Question:** To what kind of vulnerability is the application vulnerable?  
> **Answer:** `sqli` (SQL injection)

This SQLi lets us dump database data — including a password hash.

---
### Rabbit hole enumeration

CTFs love decoys. After hitting `/simple`, I spent a little time exploring other obvious paths and creds that looked promising but led nowhere useful. Those dead ends included some directories  that had no flags. I kept notes, pivoted back to the CMS exploit, and saved time — that’s the trick: try quickly, document the dead end, then move on.
<img src="/images/rabbit hole other directories.png" alt="enumeration dead end">

### Password & Cracking

I used the exploit from Exploit‑DB to pull a hashed password and the username, then cracked the hash locally with hashcat.
![[nano exploit.py.png]]
We will be using the exploit from Exploit-DB to extract a hashed password and username. 
<img src="/images/found a salt with a hash of the password.png " alt="this is the exploit.py">

we will attempt to crack this hashed password using hashcat 

<img src="/images/hash cracked and found a password for mitch.png" alt="this is the hashcat image">

>[!NOTE]  
> **Question:** What's the password?  
> **Answer:** `secret`


---

### SSH login & user flag

Use the recovered credentials to SSH into the host.

> [!NOTE]  
> **Question:** Where can you login with the details obtained?  
> **Answer:** `ssh`

<img src="/images/the first flag.png" alt="this is the first flag">

After logging in, the user flag was found inside `/usr.txt`.

> [!NOTE]  
> **Question:** What's the user flag?  
> **Answer:** `G00d j0b, keep up!`


---

### Other local users

I checked `/home` and found another user folder.

<img src="/images/the second user found.png" alt="this is the other user screenshot">

> [!NOTE]  
> **Question:** Is there any other user in the home directory? What's its name?  
> **Answer:** `sunbath`

---

### Privilege escalation (sudo / vim)

We checked allowed sudo commands to find a path to root.

```bash
sudo -l
```

The output showed the current user can run `vim` as root without a password. This can be leveraged to spawn a root shell.

> [!NOTE]  
> **Question:** What can you leverage to spawn a privileged shell?  
> **Answer:** `vim`

Example escalation command (run as the vulnerable user):

```bash
sudo vim -c ':!/bin/sh'
# or
sudo vim -c 'set shell=/bin/sh' -c ':shell'
```

<img src="/images/my user can run the vim , we'll look for the prives.png" alt="the second root flag">

---

### Root flag

After escalating to root, I read the root flag.

<img src="/images/the second flag.png" alt="this is the second flag">

> [!NOTE]  
> **Question:** What's the root flag?  
> **Answer:** `W3ll d0n3. You made it!`


---


## 🏁 Conclusion

Nice and clean box — good practice flow:

- Full port scan first — nonstandard ports matter (SSH on `2222`).
    
- Directory enumeration found `/simple` and exposed CMS Made Simple.
    
- Exploit‑DB led to `CVE-2019-9053` (SQLi) which dumped a hash.
    
- Cracked hash with hashcat → got creds → SSH → user flag.
    
- Checked `sudo -l` and used `vim` to escalate to root and get the root flag.
