---
title: UltraTech Writeup - TryHackMe
published: 2023-08-14
description: 'A walkthrough of the UltraTech room on TryHackMe, covering enumeration, web application testing, and privilege escalation.'
image: './cover.jpg'
tags: [TryHackMe, Enumeration, PrivEsc]
category: 'TryHackMe'
draft: false
---


A walkthrough of the TryHackMe UltraTech room — a grey-box style CTF focusing on enumeration, command injection, and Docker escape.

---

## Task 1: Deploy the Machine

Start this room by hitting the “Deploy” button on the right. Once the machine is deployed, you will be assigned a vulnerable machine IP. It’s a grey-box style assessment — you only have the company’s name and the server’s IP address.



## Task 2: It’s Enumeration Time

We begin with enumeration using Nmap. Below is the command used to scan all ports and detect service versions:

![Nmap Commands](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*LFsswJpwRmjg9WOQMiTgQw.png)

```bash
sudo nmap -Pn -n -sV -p- <target-ip>
```

### Questions & Answers

**1. Which software is using port 8081?**
![Q1](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*oXe7oUsX3w6zETdlM7ZVog.png)
**Ans:** Node.js

**2. Which other non-standard port is used?**
![Q2](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*ds-hVpJ-f2oN_J79jYZasA.png)
**Ans:** 31331

**3. Which software is using this port?**
![Q3](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*VIDS-wQUsT9Vaob-fWdWkA.png)
**Ans:** Apache

**4. Which GNU/Linux distribution seems to be used?**
![Q4](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*EQxk6QoTowYSmtCRAZ-BJQ.png)
**Ans:** Ubuntu

**5. The software using port 8081 is a REST API. How many of its routes are used by the web application?**
![Q5](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*sHRQx964mGzJeeq8BSh8Kw.png)
**Ans:** 2

---

## Task 3: Let the Fun Begin

We check both HTTP ports exposed in our scan.

### Port 8081

![Port 8081](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*5Qond0hdb01TfAKWxsJz3Q.png)

### Port 31331

![Port 31331](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*_k916Ens6_OlX49NVVh-Bw.png)

We check `robots.txt` on port 31331 for any disallowed paths.

![robots.txt](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*mU_ZaeacDn4baWVbX3xgzw.png)

Inside `partners.html`, we find some JavaScript functionality.

![partners.html](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*GFVkdniqexdzl9M1q-En7Q.png)

We test if command injection is possible using `ping`.

![ping](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*f-QkG_8Bd6uluGMUWzT_Fw.png)

Backticks in JavaScript evaluate commands. Using `ls` reveals the presence of a SQLite database file.

![ls command](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*r1TxbBmvfDK-1kE55y2s6Q.png)
**Filename:** `utech.db.sqlite`

We use `cat` to read it and extract password hashes.

![cat file](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*k61j0o7n_jv9ETsT7Yn9uQ.png)

Hash cracking reveals user credentials.

![hash crack](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*17N50qFeV5-EfJBRQtoOgA.png)

**2. What is the first user’s password hash?**
**Ans:** `f357a0c52799563c7c7b76c1e7543a32`

**3. What is the password associated with this hash?**
**Ans:** `n100906`

We now have SSH credentials and access to the machine as `r00t`.

---

## Task 4: The Root of All Evil

Once inside, we realize we are within a Docker container and need to escalate privileges to escape it and find the final flag.

![docker check](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*540IYp2IusIBdUKQ8DJmhA.png)

We use a method from *GTFOBins* to escape Docker and spawn an interactive shell.

![GTFOBins docker escape](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*V9YVMtDVKqJFZAqFHgO0ug.png)

Finally, we locate the private SSH key of the root user.

![id\_rsa](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*WQt2mmbI-RlZx0d8gS7qCg.png)

**What are the first 9 characters of the root user’s private SSH key?**
**Ans:** `MII******`

---

## Conclusion

This room offers a complete penetration test simulation with:

* Port & service enumeration
* Command injection via JavaScript
* Hash extraction and cracking
* Docker breakout and privilege escalation

An excellent practical exercise that highlights real-world exploitation scenarios from discovery to post-exploitation.

---
