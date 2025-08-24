---
title: Grep Writeup - TryHackMe
published: 2023-08-30
description: 'A walkthrough of the Grep room on TryHackMe, focusing on reconnaissance, OSINT techniques, and enumeration.'
image: './cover.jpg'
tags: [TryHackMe, OSINT, Recon]
category: 'TryHackMe'
draft: false
---

This Grep challenge tests your reconnaissance and OSINT skills using a vulnerable machine hosted on TryHackMe.

---

**Start this room by hitting the “deploy” button. Once deployed, you are assigned an IP address.**

Below is the Nmap command used for scanning:

![nmap\_comands](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*X3vpSoiQkX7zlK7frig1LA.png)

**Nmap Command:**

```bash
sudo nmap -n -A -Pn --min-parallelism 100 -T5 -p- <IP>
```

The scan reveals three open ports.

![nmapscan\_result](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*czgs7eYRewTGGpNN-EoByw.png)

Visit port 80 and you’ll see the default Apache page. Directory busting yields no useful results.

![default\_apachepage](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*7W_4HfBWq72OqbpSJ6NA7Q.png)

On port 443, a certificate error is displayed.

![certificate\_error](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*ganetOe__QNBBCgyO7ix7Q.png)

Inspect the certificate and find a clue: the domain `grep.thm`.

![certificate\_info](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*QjwfaYT8ozUsfKWEuHzkbQ.png)

Add an entry to `/etc/hosts` mapping the IP to `grep.thm`.

![configure](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Pyx6cyNGPLgXrGKdgPTehw.png)

Ping the domain to confirm it's properly configured.

![ping](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*nhbI48BE0GAPeVQQDdYrnw.png)

Now browse to `https://grep.thm`.

![page](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*n5l9HTAzb4tOoqK6ogAbcA.png)

Attempt to register an account.

![register\_account](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*yLlemcCp1WoxLKcxSSOSVg.png)

An error appears: "Invalid or expired API key."

![error](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*uZpf6WarUo8hs6i-Zg7_Bg.png)

Search GitHub for “SearchME grep.thm”.

![github](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*kMEPLWOmna829hs247iQ5A.png)

In the repo, locate `register.php` and find the removed API key.

![remove\_key](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*dDQAeOIwvzkCuOlOews1Bw.png)

The API key is:
`ffe60ecaa8bba2f12b43d1a4b15b8f39`

![api\_key](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*sSDpHBW0ksMXvwQ_sDEfCQ.png)

Intercept the registration request in Burp Suite, replace the API key with the correct one, and forward it.

![burp\_intercept](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*Ee_YuAUzsmx5E-goGo_uNA.png)

Registration is successful.

![successfully\_register](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*LpeNXyG3kiRigJDkO-67sg.png)

Log in to your account and retrieve the first flag.

![flag](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*-T9ZLY_Vm_yHOkXY9Awt7Q.png)

**Flag 1:** `THM{4e****************************bb}`

Check the GitHub repo again and locate `upload.php`. It validates files using hex signatures (JPG, PNG, GIF).

![code\_upload,php](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*YDD3J_l28rQ7i9-owAqZ9w.png)

Visit the `upload.php` page.

![upload.php](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*xRDMu-kxTwxwIjM1vVlsYg.png)

Use [Pentestmonkey’s PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and edit it with your IP and port.

![reverse\_hell](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*naf5CYU9jL8uP9niEkHlhw.png)

Rename the file with `.jpg` extension and adjust its hex signature to match a real image.

![change\_name](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*wmXbmZcmWiEAfR6jn4gqSw.png)

Edit magic bytes using a hex editor.

![hexvalue\_editor](https://miro.medium.com/v2/resize\:fit:1098/format\:webp/1*-DUNDw8bRgqhwe2neEYy4g.png)

Change the hex values as required.

![captionless image](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*VMUpi8mRfTj4rbBRJDj0Wg.png)

Upload the file.

![file\_uploading](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*NYWUhGHCpKHahO-K-smfxA.png)

Successful upload confirmation appears.

![file\_uploaded](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*fZJHhU0PXc2pGF376gmdag.png)

Access the uploaded file to get a reverse shell.

![upload\_folder](https://miro.medium.com/v2/resize\:fit:1236/format\:webp/1*ba5S85qfKJT7RJCrjOmT8w.png)

:::tip
If you don’t receive a reverse shell, move the `<?php` tag in the script after a comment line at the starting. This avoids corruption from hex editing the first line.
:::

With shell access, locate `users.sql` containing admin credentials.

![admin\_email](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*pCW06kF_rcHvHBLTHQ0Ilg.png)

**Admin Email:** `admin@searchme2023cms.grep.thm`

Navigate to the `leakchecker` directory and find two files. One checks for leaked credentials.

![leak\_checker](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*NtJSyjnYLu_8ozmj9_hY0w.png)

**Leak checker hostname:** `leakchecker.grep.thm`

Add this subdomain to `/etc/hosts`.

![captionless image](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*De2dqpSLLg_nBB-Znw8EjQ.png)

Go to `https://leakchecker.grep.thm`, enter the admin email, and the password is revealed.

![emailleak\_checker](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*nTs4LlEFDr-Mj5DbZycS1A.png)

**Admin Password:** `a**************!`
