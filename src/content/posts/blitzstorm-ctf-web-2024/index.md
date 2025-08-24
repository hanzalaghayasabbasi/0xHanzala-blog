---
title: Blitzstorm CTF 2024 - Web Challenge Write-Up
published: 2024-01-30
description: 'A detailed walkthrough of web challenges from Blitzstorm CTF 2024, including Tindog, Cyber-Awareness, and Discover. Covers directory busting, .git exploitation, PHP filtering, and command injection bypass.'
image: './cover.jpg'
tags: [CTF, Blitzstorm, WebSecurity]
category: 'CTF'
draft: false
---


This blog covers the web challenges I authored for **Blitzstorm CTF 2024**. Below is a walkthrough of each challenge, including exploitation techniques and bypass tricks.

---

## Challenge: Tindog

**Description:** The developer knows how to code, but he doesn’t know about security.  
**Author:** Hanzala  
**Points:** 100

After launching the challenge, we are presented with a static page about dogs.

![Tindog Page](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*rmggfeaxm1CD-uBxSGp66g.png)

Inspecting the source reveals a hidden comment.

![Comment in Code](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*7ZUEeRkrGP0kDkTGcBXRlw.png)

Navigating to `research.html` shows some research content.

![research.html](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*4DaxxDN394sSUb1BB-C0hg.png)

Using **Wappalyzer**, we identify that the backend is using **PHP**.

![Wappalyzer](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ptPtEvjcYRSTMroxvczQhA.png)

We try a PHP filter trick, such as:  
```

php-filter://resource=flag.php

```

This provides base64-encoded output of the `flag.php` source code.

![Base64 Flag](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ie6X0xU7XZMUoGXNFd_gLQ.png)

Decoding this reveals the flag.

![Decoded Flag](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*fslKVWusoQc-bNOY7lyDvA.png)

---

## Challenge: Cyber-Awareness

**Description:** This person is trying to raise awareness, but they are unaware that someone may be observing their action.  
**Author:** Hanzala  
**Points:** 100

On visiting the page, we find a basic cyber awareness-themed webpage.

![Cyber Awareness](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*5B4b4tRtLrjLQpT7QCzuwQ.png)

Nothing interesting is visible, so we run directory brute-forcing. This reveals a `.git` folder.

![.git Found](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*V8VhuMlZbOPvEPabVtWX-g.png)

We use tools like [`git-dumper`](https://github.com/arthaud/git-dumper) or download all `.git` contents manually.

![.git Download](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*2Ylq3Gx8rsl76Qhnk6DBRw.png)

After recovering the repo locally, we check the status.

![Git Status](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*wkQles1ZPWaJRTCIwB6eCA.png)

We see that the flag file was deleted. Using the following command helps us recover it:

```

git checkout --

```

![Git Checkout](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BgmsH8i660phvHB3hIHWcA.png)

We finally retrieve the flag.

![Flag](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*b1HSiT9z8jYLwxl-nC3Eog.png)

---

## Challenge: Discover

**Description:** The developer thinks this is the safest app in the world. Can you prove him wrong?  
**Author:** Hanzala  
**Points:** 200

This challenge includes a command execution feature with some filtering logic.

We observe that inserting a newline character `\n` can bypass the allowlist check and execute commands.

![Command Bypass](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*mrMd8aqHcD2hJN5obkLwRA.png)

Listing the root directory reveals the `flag` file.

![Flag Found](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*eAvgTO1ZUoKR47F1rO5Tqw.png)

However, keywords like `flag` and `.txt` are blacklisted. To bypass this, we use the following:

- `[]` character class to obfuscate characters (`f[l]ag`)
- `${IFS}` to bypass space (`cat${IFS}f[l]ag`)

This results in successful execution.

![Flag Bypass](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*SRaZeojckMQFm6J-kVj_ww.png)

---

## 🛠️ Configuration Files

The configuration files for setting up and running the above web challenges are available on GitHub.

👉 [**View the repository here**](https://github.com/hanzalaghayasabbasi/Web-CTF)


Thanks to everyone who participated — great job! 👏


