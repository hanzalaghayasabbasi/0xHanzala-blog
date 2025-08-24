---
title: Ignite CTF 2023 Writeup - Nghinx
published: 2023-12-19
description: 'A walkthrough of the Nghinx machine from Ignite CTF 2023, involving LFI exploitation and Nginx misconfiguration leading to machine takeover.'
image: './cover.jpg'
tags: [CTF, LFI, Misconfiguration]
category: 'CTF'
draft: false
---



This machine CTF demonstrates how LFI and misconfiguration in the default Nginx config can lead to machine or even account takeover.

---

###  **Challenge Details**

- **Challenge Name**: Nghinx  
- **Challenge Type**: Machine  
- **Difficulty**: Medium  
- **Points**: 100  
- **Description**:  
  _"We never been hacked, we probably have the most secure app so far."_

---

###  Initial Enumeration

When accessing the machine URL, we’re greeted with a blog homepage:

![blog-page](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*p7nDuHDQUIpXNCk5XVnSXg.png)

Clicking on "My first blog" redirects us to a `.txt` file, which throws an error:

![error-page](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*in3UnNuvANg8iftH--cmyA.png)

Let’s try removing the path from the redirection and access `/etc/passwd`:

![etc-passwd](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*sZSXb0QUrlRSLJeBxp6MgA.png)

We get a readable response. Attempting `/etc/shadow` fails due to permission issues:

![permission-error](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*wZ436qPBlH3-8x_bniWz2A.png)



### Nginx Configuration Enumeration

Given the challenge name, we suspect this is running Nginx. Trying to access `/etc/nginx/nginx.conf` doesn’t help:

![config-file](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*oYz45ygbgeHTTLXlw726fg.png)

However, reading `/etc/nginx/sites-available/default` gives us valuable information:

![default-nginx](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Eiayx3kJPczWzqHEw_BU2g.png)

#### Nginx Configuration Snippet

```nginx
server {
    listen 80;
    location ~ ^/ttydremote(.*)$ {
        proxy_pass http://127.0.0.1:7681/$1;
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
    ...
}
````

This tells us `/ttydremote` is protected by basic auth using `/etc/nginx/.htpasswd`.



### Cracking Credentials

We read the `.htpasswd` file and extract the hashed credentials:

![htpasswd](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*awW8xY7fOmQLCGx6bdESDg.png)

Use Hashcat to crack the hash:

```bash
sudo hashcat -m 1600 hash.txt /home/Ignite/Desktop/Tool/rockyou.txt
```

And we get:

* **Username**: `username`
* **Password**: `password`

![cracked-password](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*opeDSCTAo1KP8SIhU_tUew.png)


###  Gaining Access

Accessing `/ttydremote` prompts for credentials. We use the cracked ones:

![auth-login](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*l-3cz87zxR8r7hfchjpTAw.png)

Now we’re inside the system as a low-privileged user.


###  Privilege Escalation

Running `sudo -l` reveals we can execute `ansible-playbook` with sudo:

![sudo-l](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*3A_1yJ87GeMt2DOUnxgS9Q.png)

Check [GTFOBins](https://gtfobins.github.io/#ansible-playbook) for escalation steps:

![gtfobin](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*zuNkjlUauJIoxFh50NbqpQ.png)

Execute the suggested commands, gain a root shell, and grab the flag.



###  Final Flag

We navigate to `/root` and retrieve the flag:

```text
Flag{QCFAb2I1MlNaY1lld2dyUElDMVNIeU5sZz09ZjU0MjhiMmZlN2MwZmViOA==}
```

![flag](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*2WJj2bO5k62RVx10fY1ndw.png)

---

**We are done great job everyone!  👏**


