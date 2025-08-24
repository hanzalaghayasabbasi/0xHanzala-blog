---
title: Lesson Learned Writeup - TryHackMe
published: 2023-08-30
description: 'A walkthrough of the Lesson Learned room on TryHackMe, highlighting risks of unplanned SQL injection attempts and basic exploitation.'
image: './cover.jpg'
tags: [TryHackMe, SQLi, WebSecurity]
category: 'TryHackMe'
draft: false
---


This room teaches a valuable lesson: attempting SQL injection payloads without understanding their impact can lead to unintended consequences.

**Start this room by hitting the “Deploy” button on the right. Once deployed, you’ll get the vulnerable machine's IP.**

## Task 1: Find the Flag

The challenge mentions there are no rabbit holes or hidden files—just a login page and a flag.
Target: `http://MACHINE_IP/`

Navigate to the login page:

![page](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*mOxT8cQCCiag4_YZhrnkBQ.png)

I sent the login request to Burp Suite’s Intruder and tested various SQL injection payloads.
Examples:

![Intruder](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*ppH5essbU8lm5jOCqPXikg.png)

After running the attack, visiting the login page again returned a message indicating the flag had been deleted. This was caused by a destructive injection, likely using something like `OR 1=1`, which returns multiple rows and could trigger unwanted behavior in the backend logic (e.g., deleting data).
I terminated and redeployed the machine.

![lesson\_learned](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*VIATLkztcnbmBqzMtYTcOA.png)

The lesson: `OR 1=1` returns all rows, but the login logic expects only one. Instead, a safer injection that returns a single row is required.

**Working SQL Injection Payload:**

```
' UNION SELECT null-- -
```

Use any password.

![sql\_injection](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*-kmac53P8mRQU76gmUTUvw.png)

On successful login, the flag page appears and reiterates the risk of using unsafe injections in update/delete contexts.

![flag](https://miro.medium.com/v2/resize\:fit:1400/format\:webp/1*_qbHyLark28s15iATwzmDw.png)

**Flag:** `THM{a*****************************e}`

