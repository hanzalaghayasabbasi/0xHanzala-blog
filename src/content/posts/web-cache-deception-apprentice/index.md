---
title: Web Cache Deception - Apprentice | PortSwigger
published: 2024-08-29
description: 'Walkthrough of the Apprentice-level Web Cache Deception lab at PortSwigger Academy, using path mapping and static resource extension tricks to retrieve cached sensitive data.'
image: './cover.jpg'
tags: [PortSwigger,WebCache, WebSecurity]
category: 'PortSwigger'
draft: false
---

## Prerequisites

To follow this lab, you’ll need:

- A [PortSwigger Web Security Academy](https://portswigger.net/web-security) account  
- Burp Suite (Community or Professional)  
- FoxyProxy or any browser proxy setup  
- Familiarity with caching behavior and basic web protocols  

---
## Labs Covered

This write-up focuses on the following **APPRENTICE-level lab** from the PortSwigger Web Security Academy related to **Web Cache Deception**:

> **Exploiting path mapping for web cache deception**  
> This lab demonstrates how attackers can exploit path mapping weaknesses to trick web caches into storing sensitive pages.

---

### LAB 1 - Exploiting Path Mapping for Web Cache Deception

###  Lab Description

![Lab Description](https://github.com/user-attachments/assets/ab7a8cff-9b42-40ff-8e8a-65d5bc9e15c0)

---

### Overview: Exploiting Static Extensions and Path Mapping Discrepancies

Web Cache Deception attacks exploit the difference in how **origin servers** and **CDNs/caches** interpret URL paths:

- Origin servers may **ignore** extra path segments or file extensions due to RESTful routing.
- Caching layers may **treat them literally**, especially when they end with static file extensions like `.js`, `.css`, `.jpg`.

If a sensitive dynamic page (like `/my-account`) is accessed via a URL such as `/my-account/wcd.js`, the server might still respond with the user's data — but the cache sees it as a static asset and **caches it for future requests**.

This section demonstrates:
- How file extensions influence caching
- How to identify path mapping vulnerabilities
- How to store private responses in cache for exploitation


### Solution

1. **Configure FoxyProxy** to route your browser traffic through **Burp Suite**.  
   Ensure that **Intercept is turned off**, so requests are recorded in **Proxy > HTTP History**.


2: Log in to the Application 

**Log in** using the following credentials:

* **Username:** `wiener`
* **Password:** `peter`

This will authenticate you as the test user and trigger a request to the `/my-account` endpoint, which will include **Wiener’s API key** in the response.

![Login Request](https://github.com/user-attachments/assets/d019e1a0-2cb5-48ef-8a33-617dce3721dd)

3. In Burp, go to **Proxy > HTTP History**, locate the request to `GET /my-account`, and **send it to Repeater**.

![Send to Repeater](https://github.com/user-attachments/assets/13067d7c-75db-4944-91af-04b4c5d1eb9d)

4. In the **Repeater tab**, change the path to `/my-account/hanzala` and click **Send**.  
- You'll still see your API key in the response.
- This confirms the server is abstracting the path and returning user-specific data.

![Modified Path Shows Key](https://github.com/user-attachments/assets/5dd0edf8-0848-4689-a0d3-ba78fda37da1)

5. Modify the path again, this time to include a **static file extension**:  
```

/my-account/hanzala.js

````
Send the request and check the headers:

- `X-Cache: miss` → means it's not yet cached  
- `Cache-Control: max-age=30` → tells the cache to store it for 30 seconds

![X-Cache: miss](https://github.com/user-attachments/assets/18fe4e35-f28f-4ede-8343-244b1396c669)

6. **Send the same request again within 30 seconds**.  
- Now, `X-Cache: hit` will appear — confirming that the response is cached.

![X-Cache: hit](https://github.com/user-attachments/assets/d45155d7-d2bb-41ea-8b89-6bbc23869512)



### Exploit Execution

1. Go to the **Exploit Server** in your lab instance.

2. Craft an HTML payload that redirects the victim (Carlos) to a **new static-looking path** that hasn't been cached by you yet:

```html
<script>
document.location="https://YOUR-LAB-ID.web-security-academy.net/my-account/hanzalaa.js"
</script>
```
:::note
Make sure the path (`hanzalaa.js`) is different than the one you cached previously, so the victim gets a **clean cache entry**.
:::

3. Click **Deliver exploit to victim**.

![Exploit Delivery](https://github.com/user-attachments/assets/53351c6e-5477-42fd-96f7-2231f255f43b)



### Validate the Attack

1. In Burp Suite, send a request to:

   ```html
   /my-account/hanzalaa.js
   ```

2. This time, the cached response will contain **Carlos's API key**.

![Carlos API Key](https://github.com/user-attachments/assets/95ef3677-a26a-4ba5-8bda-700803c87bfa)



###  Final Step

Submit Carlos’s API key to solve the lab:

![Submit Key](https://github.com/user-attachments/assets/a1a15877-7a66-44f0-a58c-664b40b74b8c)


### Conclusion

This lab clearly demonstrates how subtle discrepancies between cache behavior and server logic can lead to severe security flaws like data leakage. Attackers can exploit these differences by crafting misleading URL paths to cache sensitive content intended for authenticated users. Always ensure your caching logic aligns with your authentication and routing mechanisms.

