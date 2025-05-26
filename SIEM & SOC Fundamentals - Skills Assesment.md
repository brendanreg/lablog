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

**Question 1:**

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

uh oh - we've got successful **network** logins from this account into DC1 and DC2. Recall that service accounts are strictly for local service management. For bonus points, let's map this to MITRE ATT&CK Framework:
Initial Access - Valid Accounts: Local Accounts (T1078.003)

<img width="452" alt="image" src="https://github.com/user-attachments/assets/de666d5b-8c88-4632-8334-24c47ac55ff2" />


The basic IRP we recieved in the prior node tells us we should consult with IT ops next.

**Question 2:**

<img width="614" alt="image" src="https://github.com/user-attachments/assets/39e3fbae-907e-4ae9-9f42-5ad1d5cab8af" />

Let's have a look as directed:

<img width="466" alt="image" src="https://github.com/user-attachments/assets/f017728d-e2a7-4234-9661-3a9a6f5fb23f" />

Only one login attempt from a singular disabled account. This isn't screaming suspicious to me. More likely a user who locked themselves out with too many bad password attempts, saw the account locked error message, and had it resolved by IT. But let's dig a bit deeper just to confirm:

<img width="698" alt="image" src="https://github.com/user-attachments/assets/9a43b847-f4dd-49eb-bde7-a5527ae9d09c" />

It turns out there are an ocean of succesful logons to DC1 with this account. This is highly suspicious, but since we don't know the nature of this account, and understand that IT operations have a habit of logging in with local admin accounts to perform administrative tasks, we'll escalate this one to tier 2/3 analysts.

**Question 3:**

<img width="618" alt="image" src="https://github.com/user-attachments/assets/dd90bc80-8841-435a-803d-fedc7adbffa1" />

Let's take a look at our custom dashboard:

<img width="464" alt="image" src="https://github.com/user-attachments/assets/6657039b-c175-4c8d-b23c-792224c8f8a7" />

Again, nothing immediately standing out, but let's triage the 3 failed attempts of Administrator on DC1:

<img width="753" alt="image" src="https://github.com/user-attachments/assets/3aea7fc2-b8df-4e1a-bfe5-d58a936f79fe" />

Many hours in between attempts & all interactive logins would indicate to me that my time is spent investigating more alarming events. Nothing suspicious.

**Question 4:**

<img width="626" alt="image" src="https://github.com/user-attachments/assets/3e1ef7c0-d39c-4b58-9fd8-2083303c7e11" />

Our lens:

<img width="465" alt="image" src="https://github.com/user-attachments/assets/d43e921c-2874-40b3-89bf-979da272aaac" />

Ah - this guy again. We've already determined there's some real sketchy activity involved with this username. We also know from the prompt that service accounts should never be initiating rdp sessions. Let's go ahead and escalate.

**Question 5:**

<img width="608" alt="image" src="https://github.com/user-attachments/assets/fc28d609-b2c6-4c58-b19a-c3b0043bff6f" />

Our dashboard:

<img width="929" alt="image" src="https://github.com/user-attachments/assets/5198fd7f-95c8-487b-b30f-7c6bd389cad8" />

Suspicious. Our local admin group was modified on PKI. Let's take a closer look and establish some context:

<img width="954" alt="image" src="https://github.com/user-attachments/assets/eb54fbd4-c01a-4d8d-8689-a0f1848dc147" />

hmm... we have a few separate occasions where administrator was added to the group. almost like it's adding and removing itself. maybe this is to evade some laps policy or detection in general.
Let's see what the administrator account is up to in the periods where it is a member of the local admin group...

<img width="688" alt="image" src="https://github.com/user-attachments/assets/486ce4ba-ce22-40ad-bf0e-8995c78b1959" />

here we see they created a powershell script. after this we see large amounts of processes being created in system32. i'm not too technically familiar with the identification of malicious processes like these, but I'm willing to bet this isn't benign. let's consult with IT operations to implement a response plan.

**Question 6:**

<img width="608" alt="image" src="https://github.com/user-attachments/assets/b76e406b-f7ce-460e-be35-e04af84ea060" />

Dashboard:

<img width="441" alt="image" src="https://github.com/user-attachments/assets/11f77b6b-1b43-4c57-b1af-f2105ac5eab5" />

Immediately we need to consult with IT operations lol. We know admin activities should only be done on PAW. 51 admin logins to another coputer is a major cause for concern.

<img width="623" alt="image" src="https://github.com/user-attachments/assets/37dffd3a-2ca7-4d89-9f86-d9ad91dfb3da" />

Dashboard:

<img width="480" alt="image" src="https://github.com/user-attachments/assets/c74c7a8f-9a2c-43ec-9258-1fad88fb3ae5" />

We know that root should never be logging in directly to linux machines based on the prompt. the only events related to root we should see are sudo escalations. This is cause for minor concern, but since the only attempts are failures, we'll just escalate this for further investigation. No need to respond immediately.








