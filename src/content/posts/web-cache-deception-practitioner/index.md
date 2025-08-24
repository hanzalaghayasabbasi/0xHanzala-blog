---
title: Web Cache Deception - Practitioner | PortSwigger
published: 2024-09-10
description: >
  Practitioner-level Web Cache Deception labs from PortSwigger Academy covering
  path delimiter abuse, normalization discrepancies in origin and cache servers, and
  how these mismatches lead to private data exposure via cache poisoning.
tags: [PortSwigger, WebCache ,WebSecurity]
category: 'PortSwigger'
image: './cover.jpg'
draft: false
---

## Prerequisites

To follow along, ensure you have:

* A [PortSwigger Academy](https://portswigger.net/web-security) account 
* Burp Suite (Community or Professional)
* FoxyProxy (or equivalent proxy setup)
* JavaScript enabled in your browser

# Web Cache Deception: Exploiting Path Delimiters and Normalization (Practitioner Labs)

This blog post explores **Web Cache Deception**, a vulnerability arising from discrepancies in URL processing between web servers and caching layers. We’ll cover three **Practitioner-level labs** from [PortSwigger’s Web Security Academy](https://portswigger.net/web-security/web-cache-deception) that demonstrate advanced techniques for exploiting path delimiters, origin server normalization, and cache server normalization.

---

## Table of Contents
- [Web Cache Deception: Exploiting Path Delimiters and Normalization (Practitioner Labs)](#web-cache-deception-exploiting-path-delimiters-and-normalization-practitioner-labs)
  - [Table of Contents](#table-of-contents)
  - [Introduction to Web Cache Deception](#introduction-to-web-cache-deception)
  - [Lab 1: Exploiting Path Delimiters for Web Cache Deception](#lab-1-exploiting-path-delimiters-for-web-cache-deception)
    - [Lab Description](#lab-description)
    - [Technical Overview](#technical-overview)
    - [Solution](#solution)
  - [Lab 2: Exploiting Origin Server Normalization for Web Cache Deception](#lab-2-exploiting-origin-server-normalization-for-web-cache-deception)
    - [Lab Description](#lab-description-1)
    - [Technical Overview](#technical-overview-1)
    - [Solution](#solution-1)
  - [Lab 3: Exploiting Cache Server Normalization for Web Cache Deception](#lab-3-exploiting-cache-server-normalization-for-web-cache-deception)
    - [Lab Description](#lab-description-2)
    - [Technical Overview](#technical-overview-2)
    - [Solution](#solution-2)
  - [Conclusion](#conclusion)


---

## Introduction to Web Cache Deception

**Web Cache Deception** occurs when attackers manipulate URL paths to trick cache servers into storing sensitive, user-specific content (e.g., API keys, user profiles) as static resources (e.g., `.css` or `.js` files). This vulnerability stems from mismatches in how **origin servers** and **cache servers** handle URL paths, particularly involving **delimiters**, **encoded characters**, and **path normalization**.

The labs covered demonstrate:
- **Path Delimiter Exploitation**: Using characters like `;` or `?` that are ignored by the origin server but treated literally by the cache.
- **Origin Server Normalization**: Exploiting differences in resolving encoded path segments (e.g., `..%2f`).
- **Cache Server Normalization**: Leveraging cache-specific decoding of paths to cache sensitive content.

---

## Lab 1: Exploiting Path Delimiters for Web Cache Deception

### Lab Description
This lab shows how attackers can abuse **path delimiter variations** (e.g., `;` or `?`) to trick caches into storing sensitive content under static-looking paths.

![Lab Description](https://github.com/user-attachments/assets/bd95fe48-98e4-41d2-b70a-ccff4ad363b3)

### Technical Overview
Delimiter discrepancies arise when the **origin server** and **cache server** interpret special characters differently:
- **Framework Behavior**:
  - `;` (semicolon) is used for matrix parameters in Java Spring.
  - `.` (dot) denotes response format in Ruby on Rails.
  - `%00` (null byte) acts as a terminator in OpenLiteSpeed.
- **Cache Behavior**: Caches treat these characters literally, assuming they’re part of a static file path (e.g., `/account;foo.css`).

**Attack Flow**:
1. Craft a URL like `/my-account;fake.css`.
2. The origin server ignores `;fake.css` and returns sensitive data (e.g., user profile).
3. The cache, seeing `.css`, stores the response as a static resource.
4. Attackers access the cached response at `/my-account;fake.css`.

**Delimiter Decoding**:
- Encoded delimiters like `%23` (`#`) or `%3F` (`?`) can exploit decoding mismatches.
- Example: `/profile%23wcd.css` → Cache sees `.css`, origin decodes `%23` to `#` and processes `/profile`.

**Exploitation Steps**:
- Use **Burp Intruder** to test delimiters (`;`, `.`, `!`, `%00`, etc.) in raw and encoded forms.
- Check response headers (e.g., `X-Cache`) and content to confirm caching of dynamic responses.

### Solution
1. Configure FoxyProxy to route traffic through Burp Suite. Disable Intercept and log in as `wiener:peter`.
   ![Login Screen](https://github.com/user-attachments/assets/94d8aa76-292b-4e66-9656-c7af5fc545a2)
2. In **Proxy > HTTP History**, locate `GET /my-account`. Right-click and send to **Repeater**.
   ![HTTP History](https://github.com/user-attachments/assets/a61315fa-15f0-40fd-8618-8d91d0eed97b)
3. In Repeater, test `/my-account/abc` → **404 Not Found** (no caching).
   ![404 Response](https://github.com/user-attachments/assets/cba05710-fe19-43f2-b8cb-9b6842760779)
4. Test `/my-accountabc` → **404 Not Found** (baseline for delimiter testing).
   ![404 Response](https://github.com/user-attachments/assets/e2026f55-157e-4502-be8f-d62a9bf19e3a)
5. Send `/my-account` to **Intruder**. Set payload position as `/my-account§§abc` and use [PortSwigger’s delimiter list](https://portswigger.net/web-security/web-cache-deception/wcd-lab-delimiter-list).
   ![Intruder Setup](https://github.com/user-attachments/assets/d9490ea4-5a20-4ff8-8f68-20b4cfa8cb8e)
6. Disable **URL-encode** in Payload settings.
   ![Payload Encoding](https://github.com/user-attachments/assets/34350a63-4f9b-49ce-98b7-f34fb4b1988e)
7. Run the attack. Note `;` and `?` return **200 OK** with the API key, indicating they are delimiters.
   ![Intruder Results](https://github.com/user-attachments/assets/4ef5c644-ed0c-437a-91fa-6b75faa49c8e)
8. In Repeater, test `/my-account?abc.js` → **200 OK**, no caching (cache uses `?` as a delimiter).
   ![Repeater Response](https://github.com/user-attachments/assets/43705e23-55cd-4e82-86e0-e9a1696741cb)
9. On the **Exploit Server**, craft an exploit to redirect user `carlos` to `/my-account;hanzala.js`:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account;hanzala.js"</script>
   ```
   ![Exploit Server](https://github.com/user-attachments/assets/4f3d2b87-1726-4d8a-b4a2-1c5a46498a2e)
10. Deliver the exploit. Revisit `/my-account;hanzala.js` in Repeater to retrieve Carlos’s API key.
    ![API Key Response](https://github.com/user-attachments/assets/35a85ccd-f204-44f1-a807-fbe7280ecf15)
11. Submit the API key to complete the lab.
    ![Lab Completion](https://github.com/user-attachments/assets/ff895cd1-2748-4283-855c-bdd032904b58)

---

## Lab 2: Exploiting Origin Server Normalization for Web Cache Deception

### Lab Description
This lab demonstrates how mismatches in **URL normalization** between the origin and cache servers allow attackers to cache sensitive content under static directories using encoded dot-segments (e.g., `..%2f`).

![Lab Description](https://github.com/user-attachments/assets/3b815827-987e-4604-b53f-7d6382905e6b)

### Technical Overview
**Static directory cache rules** (e.g., `/resources`, `/static`) instruct caches to store responses as static resources. If the **origin server** resolves encoded paths (e.g., `/resources/..%2fprofile` to `/profile`) differently from the cache, attackers can:
1. Craft a URL matching the cache’s static rule (e.g., `/resources/..%2fmy-account`).
2. The origin server resolves it to a sensitive endpoint (`/my-account`).
3. The cache stores the response under the static path.

**Example**:
```
/resources/..%2fprofile
→ Cache: Matches /resources rule, caches response
→ Origin: Resolves to /profile, returns sensitive data
```

**Exploitation Steps**:
- Use **Burp Suite** to test path normalization with encoded dot-segments (`..%2f`).
- Check for `X-Cache` headers to confirm caching behavior.

### Solution
1. Configure FoxyProxy and Burp Suite. Log in as `wiener:peter`.
   ![Login Screen](https://github.com/user-attachments/assets/516ebcad-7a08-47eb-a165-be60f038f2d7)
2. In **Proxy > HTTP History**, note `/resources` paths are cached. Send a `/resources` request to **Repeater**.
   ![HTTP History](https://github.com/user-attachments/assets/af6154f5-913a-4eed-83f9-27cadf02815f)
3. Test `/resources/..%2fRESOURCES` → **404 Not Found** with `X-Cache: miss`.
   ![404 Response](https://github.com/user-attachments/assets/1c935974-79d9-4022-9303-c129d1212400)
4. Resend → `X-Cache: hit`, indicating cache doesn’t decode dot-segments.
   ![Cached Response](https://github.com/user-attachments/assets/7feaccd7-5131-446e-abe2-e6765d5f95b6)
5. Test `/resources/..%2fmy-account` → **200 OK** with API key and `X-Cache: miss`.
   ![200 Response](https://github.com/user-attachments/assets/4677cc79-8d39-471a-ad48-b3d3fae49489)
6. Resend → `X-Cache: hit`.
   ![Cached Response](https://github.com/user-attachments/assets/a3536c13-afed-4387-a046-42fcfc33a765)
7. On the **Exploit Server**, craft an exploit:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/resources/..%2fmy-account"</script>
   ```
   ![Exploit Server](https://github.com/user-attachments/assets/d68fb152-f190-487a-b98d-5bea204fbead)
8. Deliver the exploit. Visit `/resources/..%2fmy-account` to retrieve Carlos’s API key.
   ![API Key Response](https://github.com/user-attachments/assets/15832a16-a7f4-4eb1-a45a-c299ea272d31)
9. Submit the API key to complete the lab.
   ![Lab Completion](https://github.com/user-attachments/assets/a864d670-e78b-4ce8-bcb9-127d84fb8c2c)

---

## Lab 3: Exploiting Cache Server Normalization for Web Cache Deception

### Lab Description
This lab explores how **cache server normalization** discrepancies, particularly in decoding encoded dot-segments (e.g., `%2f%2e%2e%2f`), allow caching of sensitive content under static paths.

![Lab Description](https://github.com/user-attachments/assets/101cf7b5-0785-42a7-a336-785a24ce8928)

### Technical Overview
If the **cache server** decodes paths (e.g., `/my-account%2f%2e%2e%2fresources` to `/resources`) but the **origin server** truncates at a delimiter (e.g., `%23` for `#`), attackers can:
1. Craft a URL that the cache maps to a static directory.
2. The origin server processes it as a sensitive endpoint.
3. The cache stores the response under the static path.

**Example**:
```
/my-account%23%2f%2e%2e%2fresources
→ Cache: Resolves to /resources, caches response
→ Origin: Truncates at %23, processes /my-account
```

**Exploitation Steps**:
- Test encoded dot-segments and delimiters (`%23`, `%3f`) to identify cache vs. origin behavior.
- Use `X-Cache` headers to confirm caching.

### Solution
1. Configure FoxyProxy and Burp Suite. Log in as `wiener:peter`.
   ![Login Screen](https://github.com/user-attachments/assets/14ebcd29-bd03-4f8b-89a9-329b3703eb18)
2. In **Proxy > HTTP History**, note caching for `/resources` paths. Send a `/resources` request to **Repeater**.
   ![HTTP History](https://github.com/user-attachments/assets/1f338b89-bb86-4a97-bfd5-ba6a8f8d17b7)
3. Test `/aaa/..%2fresources/YOUR-RESOURCE` → **404 Not Found** with `X-Cache: miss`.
   ![404 Response](https://github.com/user-attachments/assets/be50d631-5f81-4c46-9347-e4027064c997)
4. Resend → `X-Cache: hit`, confirming cache resolves dot-segments.
   ![Cached Response](https://github.com/user-attachments/assets/bf902fca-8006-4566-b7fc-121d17d86393)
5. Test `/resources/.%2fYOUR-RESOURCE` → **404 Not Found**, no caching.
   ![404 Response](https://github.com/user-attachments/assets/66700a78-6da3-44cf-a7cc-44d1bf322ccd)
6. Test `/my-account%2f%2e%2e%2fresources` → **200 OK** with API key, no caching.
   ![200 Response](https://github.com/user-attachments/assets/11261156-c3cd-48f2-91d6-ed0436d2e647)
7. Test `/my-account%23%2f%2e%2e%2fresources` → **200 OK** with `X-Cache: miss`.
   ![200 Response](https://github.com/user-attachments/assets/6e3ceef6-36e1-43fe-8448-442bb481ceda)
8. Resend → `X-Cache: hit`.
   ![Cached Response](https://github.com/user-attachments/assets/e8aeddbc-d4d5-4bf9-91e8-b4ce8db6ad10)
9. On the **Exploit Server**, craft an exploit:
   ```html
   <script>document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account%23%2f%2e%2e%2fresources?hanzala"</script>
   ```
   ![Exploit Server](https://github.com/user-attachments/assets/57476d36-47d1-4a12-b44f-df946094d326)
10. Deliver the exploit. Visit `/my-account%23%2f%2e%2e%2fresources?hanzala` to retrieve Carlos’s API key.
    ![API Key Response](https://github.com/user-attachments/assets/f92082e9-fa7c-40cf-a3e0-bca10d3bb56f)
11. Submit the API key to complete the lab.
    ![Lab Completion](https://github.com/user-attachments/assets/a78ed3de-8899-4c5a-a25b-d0042200d044)

## Conclusion

These Practitioner-level labs demonstrate how subtle inconsistencies in how cache servers and origin servers interpret URLs can lead to serious vulnerabilities. By exploiting differences in **delimiter parsing**, **path normalization**, and **encoded character handling**, attackers can trick caching layers into storing and serving sensitive user data.

As a developer or security engineer, it's critical to:

* Ensure authentication checks are not bypassable via alternative path encodings.
* Prevent caching of user-specific responses under any circumstances.
* Understand how your CDN and application framework parse and normalize paths.

Even seemingly innocuous URL structures can have severe consequences if caching layers and backend logic fall out of sync. Proper cache controls, strict routing logic, and thorough testing are essential defenses against these attacks.


