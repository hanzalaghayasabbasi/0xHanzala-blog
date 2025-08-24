
---
title: Web Cache Deception - Expert | PortSwigger
published: 2024-09-12
description: 'Walkthrough of the Expert-level Web Cache Deception lab from PortSwigger Academy, focusing on exploiting exact-match cache rules using advanced path normalization techniques.'
image: './cover.jpg'
tags: [PortSwigger, WebCache , WebSecurity]
category: 'PortSwigger'
draft: false
---

# Web Cache Deception – Exploiting Exact-Match Cache Rules (Expert Level)

## Prerequisites

To complete this lab, ensure you have:

* A [PortSwigger Web Security Academy](https://portswigger.net/web-security) account
* Burp Suite (Community or Professional edition)
* A proxy extension like FoxyProxy configured in your browser
* JavaScript enabled in your browser
* Familiarity with web caching mechanisms, HTTP, HTML, and CSRF concepts

---

## Labs Covered

This write-up focuses on the following **EXPERT-level lab** from the PortSwigger Web Security Academy:

**Lab: Exploiting exact-match cache rules for web cache deception**
This lab demonstrates how attackers can abuse exact-match cache rules to trick caches into storing sensitive resources.

---

## Lab Description

![Lab Description](https://github.com/user-attachments/assets/757cc6d4-4764-4097-bf87-83fb43ae352c)

---

## Overview: Exploiting File Name Cache Rules

Certain resources like `robots.txt`, `favicon.ico`, and `index.html` are often cached using exact-match filename rules. An attacker can exploit discrepancies in how URLs are interpreted by the cache vs the origin server to expose or cache sensitive user data.



## Detecting Normalization Discrepancies

To understand whether the cache and origin server treat paths differently:

* Try requests like `/aaa%2f%2e%2e%2frobots.txt`
* Observe caching headers (`X-Cache: hit/miss`)

If the origin doesn’t normalize paths, but the cache does, this can be abused.

---

## Exploiting the Lab

### Step 1 – Identify Sensitive Endpoints

Login with credentials `wiener:peter` and change your email address.

![Login Interface](https://github.com/user-attachments/assets/be70e9f3-21d8-4c07-9c35-903d9b648447)

Observe the CSRF token in `/my-account`.

![CSRF Token in /my-account](https://github.com/user-attachments/assets/9114f8cc-bd8c-436c-a5b0-45aa616e099f)


### Step 2 – Investigate Path Discrepancies

Test variants of `/my-account` using unusual path syntax:

* `/my-account/hanzala`
* `/my-accounthanzala`

![404 Responses](https://github.com/user-attachments/assets/bbb18c69-3c31-4d68-a0e8-49fc10b3f1d6)
![Also 404](https://github.com/user-attachments/assets/fee10535-45bf-4537-8647-c62b72031289)

Run Intruder tests using various delimiters like `;`, `?`, `%2f`.

![Intruder Setup](https://github.com/user-attachments/assets/4d42917f-d4f1-4812-8042-ed6180f69494)
![Disable Payload Encoding](https://github.com/user-attachments/assets/f0ab261a-5995-448a-8723-22847d43ce78)

Observe responses. Note which variants return 200.

![200 Responses on ; and ?](https://github.com/user-attachments/assets/bfe24e8e-13b3-4afa-b5b0-5d801510d134)


### Step 3 – Check Caching Behavior

Requests such as:

* `/my-account;hanzala.js`
* `/my-account?hanzala.js`

Return 200 but show **no cache**.

![No Cache Evident](https://github.com/user-attachments/assets/36b964c8-6b8d-4c1d-bd8b-40173d19d9f6)



### Step 4 – Test Normalization Discrepancy

* Try `/aa/..%2fmy-account` → 404
  Origin doesn't resolve dot-segments.

![Dot Segment Fails](https://github.com/user-attachments/assets/2e4dc716-dc96-4df9-8ac3-77815beaeffb)

Now test `/robots.txt`:

* First request shows `X-Cache: miss`
* Second request shows `X-Cache: hit` (indicates cache)

![robots.txt Caching](https://github.com/user-attachments/assets/e6592c65-eb4f-4443-8dd0-fa355af2717a)
![Now a Hit](https://github.com/user-attachments/assets/6b6fb2f2-4171-4d59-9176-4c6017aa6fed)

Try `/aaa/..%2frobots.txt`:

![Cache Hit on Normalized Path](https://github.com/user-attachments/assets/9306a0d6-18da-4d4b-894c-63590584fa1b)



### Step 5 – Poison the Cache

Try a crafted request:

* `/my-account;%2f%2e%2e%2frobots.txt`

1st Request:

![First Miss](https://github.com/user-attachments/assets/bf0f05d8-0033-4b7f-b4ee-dce2a7adfee4)

Resend:

![Now a Cache Hit](https://github.com/user-attachments/assets/675c8744-a3f1-4553-bdc3-02c833bdd853)

The response includes sensitive data (your CSRF token), now cached.


### Step 6 – Deliver Exploit to Victim

On the exploit server:

```html
<img src="/my-account;%2f%2e%2e%2frobots.txt?wc" />
```

![Exploit Payload Delivery](https://github.com/user-attachments/assets/7aaf0ece-38f8-4925-a30c-e568643f27c4)

When the administrator visits this image:

* They load your `/my-account;%2f%2e%2e%2frobots.txt?wc` path
* Their CSRF token gets stored in cache

Verify this by resending the same request:

![Token Appears](https://github.com/user-attachments/assets/2890699e-ee32-4fd7-9b4b-7772af7e4235)



### Step 7 – Craft and Send CSRF Exploit

In Burp, copy the request to change email:

![Change Email Request](https://github.com/user-attachments/assets/a9f8ebe3-caaa-4507-b0be-62f07a32a34b)

Replace your token with the stolen one from cache.

Generate CSRF HTML:

![Generate CSRF PoC](https://github.com/user-attachments/assets/3d0c8378-1706-4e5f-b9fd-56b9695439c9)

Copy HTML and paste into exploit body:

![Paste HTML](https://github.com/user-attachments/assets/b6dec4e3-5501-42b4-911c-c4573b8c2847)

Deliver the payload again:

![Exploit Delivered](https://github.com/user-attachments/assets/7b205cc4-d0d2-471a-b9ca-8bb6b63119ce)

Confirm lab is solved.

![Lab Solved](https://github.com/user-attachments/assets/bf0ae86d-1cfc-4c36-b6ad-31491f55cf3d)



## Conclusion

This lab demonstrates how a mismatch between the origin and cache handling of exact-match filenames can be exploited to store and reuse user-specific content. By targeting cacheable endpoints such as `robots.txt`, and tricking the cache into storing personalized responses, an attacker can stage highly effective CSRF attacks.

