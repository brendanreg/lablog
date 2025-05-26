Below is a write-up for the skills assesment section of the SIEM & SOC fundamentals module in HTB's SOC Analyst learning path.
This module covers the foundational terminology and concepts you must be intimately familiar with in order to be an effective SOC analyst.
In addition, it covers how we can leverage SIEM solutions, particularly the ELK stack, to triage events and incidents.
Throughout the module, we've compiled an array of custom visualizations that allow us to keep an eye on potential threats, and performed deeper analysis on some of the more suspicious alerts with the discover feature.
The skills assesment will test our ability to perform as an analyst in a hands-on simulated environment.
With all that being said, let's jump into the assesment:

**Prompt:**
Congratulations,

You have been hired in Eagle as a SOC Tier 1 analyst. Yesterday was your on-boarding day with the company, and today you will be familiarized with the SOC. Your day will begin by meeting up with a senior analyst, who will provide insights into the environment, and afterwards, you are expected to begin monitoring alerts and security events in our home-cooked SOC dashboards.

The following are your notes after meeting the senior analyst, who provided insights into the environment:

The organization has moved all hosting to the cloud; the old DMZ network is closed down, so no more servers exist there.

The IT operation team (the core IT admins) consists of four people. They are the only ones with high privileges in the environment.

The IT operation team often tends to use the default administrator account(s) even if they are told otherwise.

All endpoint devices are hardened according to CIS hardening baselines. Whitelisting exists to a limited extent.

IT security has created a privileged admin workstation (PAW) and requires that all admin activities be performed on this machine.

The Linux environment is primarily 'left over' servers from back in the day, which have very little, if any, activity on a regular day. The root user account is not used; due to audit findings, the account was blocked from connecting remotely, and users who require those rights will need to escalate via the sudo command.

Naming conventions exist and are strictly followed; for example, service accounts contain '-svc' as part of their name. Service accounts are created with long, complex passwords, and they perform a very specific task (most likely running services locally on machines).

<img width="618" alt="image" src="https://github.com/user-attachments/assets/0f97b9aa-e9fc-4ec0-8ce8-0b3057e48cb9" />

Let's jump into our custom dashboard as directed and take a look at our alerts:

<img width="467" alt="image" src="https://github.com/user-attachments/assets/2bb4de4c-7c34-40bc-9dce-3dfa4301b20b" />

As we can see, nothing immediately jumps out. We don't see a high volume of failed login attempts, and admin accounts are only attempting to log in to domain controllers/PAW. However, one thing does raise an eyebrow; sql-svc1 has attempted to login to PKI. We know from the notes provided to us that service accounts perform a specific function, running services locally on a machine. But the behavior we see here doesn't match that use case. Let's jump to the discover tab and try to establish some context. First let's see if we have a record of any succesful logon events by the account in question:

<img width="946" alt="image" src="https://github.com/user-attachments/assets/eba37bd1-056f-437c-8421-0c67c509df95" />

It appears the only logon attempts we have are the two failed ones we were alerted of on the dashboard. Let's check the substatus to identify the reason of failure for the logins:

<img width="551" alt="image" src="https://github.com/user-attachments/assets/de8fd7a9-45b1-419c-bb08-a47f6db4cdd8" />

username doesn't exist... okay, let's see if it's similar to any other usernames.

<img width="951" alt="image" src="https://github.com/user-attachments/assets/56e0a288-829a-478f-81c0-ae5196387790" />

aha! there's a similar username with several login events. let's take a closer look at this account and see where it takes us:

<img width="948" alt="image" src="https://github.com/user-attachments/assets/bbed6624-58f0-48cb-86e3-c709e6a09117" />

uh oh - we've got successful **network** logins from this account into DC1 and DC2. Recall that service accounts are strictly for local service management.
The basic IRP we recieved in the prior node tells us we should consult with IT ops next.











