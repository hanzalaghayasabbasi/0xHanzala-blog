---
title: Blaster Writeup - TryHackMe
published: 2023-08-20
description: 'A TryHackMe Blaster room walkthrough focusing on enumeration, vulnerability exploitation, and privilege escalation.'
image: './cover.jpg'
tags: [TryHackMe, Exploitation, PrivEsc]
category: 'TryHackMe'
draft: false
---

## Task 1 : Mission Start!


Start this room by hitting the “deploy” button on the right!**Once you have deployed machine you have assigned a Vulnerable machine IP**.It is a **grey-box** kind of assessment, the only information you have is the company’s name and their server’s IP address.

## Task 2 : Activate Forward Scanners and Launch Proton Torpedoes

These are some command of nmap that we are going to used in our nmap scan:-

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*YHfPnUU6_EaaN5IaE4-kvQ.png)

First we ping our host , but it’s showing host is down , So then we performed tcp host discovery scan it’s showing us **host up**.

![nmap_hostscan](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*mTZ1zNR1PIJ0n3LTcXbveA.png)

Then we performed full port scan of nmap.

![nmap_portscan](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*J7zxsEqbewcu6EJ7zbOmow.png)

**1 How many ports are open on our target system?**

**Ans** : 2.

**2 :Looks like there’s a web server running, what is the title of the page we discover when browsing to it?**

**Ans**: IIS Windows Server.

So we have discoverd port 80 is open , Now we will do **Directory bursting.**

![ffuf](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OfPtRfwKMWpPjnUdDEIe-w.png)

From the ffuf result ,we have find two directories , So we traverse to first directory.

![traversing_retro](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*AkrNe4Y4_vWYsBl8SHkuzg.png)

Now we have look at first directory , So we go to second directory and notice that both were same.

![traversing_Retro](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*e8cvudE5QO0hEWun9nVVog.png)

**3 Interesting, let’s see if there’s anything else on this web server by fuzzing it. What hidden directory do we discover?**

**Ans:** /retro or /Retro.

We have notice that all the paragraph are written by only one person and at second last paragraph, that is giving us passwords hint.

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*mPhBFW8NgYeHpt3nhaMTGw.png)

**4 Navigate to our discovered hidden directory, what potential username do we discover?**

**Ans :** wade.

Now we search for hint on the google.

![Google:search](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*o0Hw5SSFLyVW8b2XgyoJpg.png)

**4 Crawling through the posts, it seems like our user has had some difficulties logging in recently. What possible password do we discover?**

**Ans:** parzival

Now we get user name and password and Remote desktop protocol is opens , So we try to connect to machine.

![rdp_login](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*NPuNgJoymwuoBVV4yh5sKw.png)

We have successfully login on the machine using credential that we have find.

![login_succefully](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*5xxZyDDUtp7AMAH0_VsWbA.png)

The user flag is on the Desktop in the file name user.txt.

![flag](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*TShQYYrybNCMsf9UBv6Tbg.png)

**5 Log into the machine via Microsoft Remote Desktop (MSRDP) and read user.txt. What are it’s contents?**

**Ans:** THM{H*************E}

---

## Task 3 : Breaching the Control Room


We have Notice there is some executable tool or software is placed on the Desktop.

![login_succefully](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*gdDhNb9PU4H2q-HpiyOyWw.png)

We search for that name on the google and we get the result show in the below image:-

![CVE](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*WVfGESy6P_1PNHLCaw-oWw.png)

**1 When enumerating a machine, it’s often useful to look at what the user was last doing. Look around the machine and see if you can find the CVE which was researched on this server. What CVE was it?**

**Ans:** CVE-2019–1388.

**2 Looks like an executable file is necessary for exploitation of this vulnerability and the user didn’t really clean up very well after testing it. What is the name of this executable?**

**Ans:** hhupd.

Now we search for [**CVE-2019–1388**](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-1388) in google, we come to know that when hhupd run as admin and the prompt show Certificate option which is validate that person is admin or not by looking at password it is bypassing it.

First Click on hhupd exploit and run as administrator and then click on show information about the publisher certificate.

![runas_admin](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*n8VIg0iqgUnaTK_VIGzbgw.png)

Click on Versingn Commercial Software Publisher and click ok.

![Issued-by](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*P8hYavg0n03I1Rc6r6TqKw.png)

Then a browser will open in front of you have to save the page and then click on ok option.

![saves_as](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*70xJr9PgNu2aT6OzgjGsig.png)

Save Page at **C:\Windows\System32\\** and the click save option.

![Save_location](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*nrMfa4ZrhuJH4yLUA_vxQw.png)

Now search for cmd in the location you have save page and right click and open cmd.

![open_cmd](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*EIZ51Hir9WHrLt0reGw5Kw.png)

A terminal will appear in front of you click whoami.

![Privilige_escalation](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*AOwlhy9nTQ0xkg9KXTPlpA.png)

Congratulations, You have done Privilege Escalation.

**3 Now that we’ve spawned a terminal, let’s go ahead and run the command ‘whoami’. What is the output of running this?**

**Ans:** nt authority\system

![captionless image](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BEFiEtoDDxTCWv8FR_nYSg.png)

**4 Now that we’ve confirmed that we have an elevated prompt, read the contents of root.txt on the Administrator’s desktop. What are the contents? Keep your terminal up after exploitation so we can use it in task four!?**

**Ans :** THM{C************************N}

---

## Task 4 : Adoption into the Collective


Now it’s time to connect Vulnerable machine which have admin Privileges to our local machine, So we used exploit [**multi/script/web_delivery**](https://www.offsec.com/metasploit-unleashed/web-delivery/) and set target to two.

![target_language](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*L1Of9pUfZA8QLE-q-bcw_Q.png)

**1 First, let’s set the target to PSH (PowerShell). Which target number is PSH?**

**Ans:** 2

The below images show setting , we have done in our exploit [**multi/script/web_delivery**](https://www.offsec.com/metasploit-unleashed/web-delivery/).

![setting_exploit](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*R6zhXwdKH36fKOdkuZjfZA.png)

The payload which is generated by ou exploit copy it and paste to admin privilege terminal , So it will connect back to our metasploit.

![payload](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xSlbV4UMk1EpPYYR9PgOWQ.png)

So payload connect back to the metasploit

![connection_successfull](https://miro.medium.com/v2/resize:fit:1950/format:webp/1*8g2l8YMCYyXTlCru0v8JaA.png)

Payload create a reverse shell on metasploit , So if we do whoami it‘s showing us **nt authority\system.**

![whoami](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*okYJfHZ7OBE1v_sijGak8A.png)

So we look for persistence mechanisms in the metaspliot.

![persistence](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*kt2yKk0UCKZQvuS9n4SX5w.png)

**2 Last but certainly not least, let’s look at persistence mechanisms via Metasploit. What command can we run in our meterpreter console to setup persistence which automatically starts when the system boots? Don’t include anything beyond the base command and the option for boot startup.**

**Ans :** run persistence -X

