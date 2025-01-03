---
icon: material/text-box-outline
---

# Elf Stack

**Official Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star::fontawesome-solid-star:<br/>

**My Difficulty Rating**:
:fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-solid-gift::fontawesome-regular-square:<br/>

## Objective

!!! question "Request"
    Help the ElfSOC analysts track down a malicious attack against the North Pole domain.

??? quote "Fitzy Shortstack"
    Greetings! I'm the genius behind the North Pole Elf Stack SIEM. And oh boy, we've got a situation on our hands.

    Our system was attacked—Wombley's faction unleashed their FrostBit ransomware, and it's caused a digital disaster.

    The logs are a mess, and Wombley's laptop—the only backup of the Naughty-Nice List—was smashed to pieces.

    Now, it's all up to you to help me trace the attack vectors and events. We need to figure out how this went down before it's too late.

    You'll be using a containerized ELK stack or Linux CLI tools. Sounds like a fun little puzzle, doesn't it?

    Your job is to analyze these logs... think of it as tracking snow tracks but in a digital blizzard.

    If you can find the attack path, maybe we can salvage what's left and get Santa's approval.

    Santa's furious at the faction fighting, and he's disappointed. We have to make things right.

    So, let's show these attackers that the North Pole's defenses are no joke!


## Hints

??? tip "Elf Stack Intro"
    I'm part of the ElfSOC that protects the interests here at the North Pole. We built the Elf Stack SIEM, but not everybody uses it. Some of our senior analysts choose to use their command line skills, while others choose to deploy their own solution. Any way is possible to hunt through our logs!

??? tip "Elf Stack Fields"
    If you are using your command line skills to solve the challenge, you might need to review the configuration files from the containerized Elf Stack SIEM.

??? tip "Elf Stack Hard - Email 1"
    I was on my way to grab a cup of hot chocolate the other day when I overheard the reindeer talking about playing games. The reindeer mentioned trying to invite Wombley and Alabaster to their games. This may or may not be great news. All I know is, the reindeer better create formal invitations to send to both Wombley and Alabaster.

??? tip "Elf Stack Hard - Email 2"
    Some elves have tried to make tweaks to the Elf Stack log parsing logic, but only a seasoned SIEM engineer or analyst may find that task useful.

??? tip "Elf Stack WinEvent"
    One of our seasoned ElfSOC analysts told me about a great resource to have handy when hunting through event log data. I have it around here somewhere, or maybe it was online. Hmm.

??? tip "Elf Stack PowerShell"
    Our Elf Stack SIEM has some minor issues when parsing log data that we still need to figure out. Our ElfSOC SIEM engineers drank many cups of hot chocolate figuring out the right parsing logic. The engineers wanted to ensure that our junior analysts had a solid platform to hunt through log data.

## Solution

After downloading the three files, we must extract the elf-stack-siem and then move the two log 
zips into its folder. Run
```
docker compose up setup
docker compose up
```

We can then open http://localhost:5601/app/home and use the credentials elastic / ELFstackLogin!
to log in (as found in the .env file)

### Easy

* Question 1: How many unique values are there for the event_source field in all logs?

    Analytics > Dashboard > Create new > Create visualisation > Drag event_source.

    This shows that there are *5*

* Question 2: Which event_source has the fewest number of events related to it?

    Shown in previous view: *AuthLog*

* Question 3: Using the event_source from the previous question as a filter, what is the field name that contains the name of the system the log event originated from?

    Go to Analytics > discover > event_source : "AuthLog" 

    *event.hostname*

* Question 4: Which event_source has the second highest number of events related to it?

    Use the first analytics dashboard to see it's *NetflowPmacct*

* Question 5: Using the event_source from the previous question as a filter, what is the name of the field that defines the destination port of the Netflow logs?

    Go to Analytics > discover > event_source : "NetflowPmacct"
    
    *event.port_dst*

* Question 6: Which event_source is related to email traffic?

    We can guess this based on the event sources we saw in question one: *SnowGlowMailPxy*

* Question 7: Looking at the event source from the last question, what is the name of the field that contains the actual email text?

    *event.Body*

* Question 8: Using the 'GreenCoat' event_source, what is the only value in the hostname field?

    *SecureElfGwy*

* Question 9: Using the 'GreenCoat' event_source, what is the name of the field that contains the site visited by a client in the network?

    *event.url*

* Question 10: Using the 'GreenCoat' event_source, which unique URL and port (URL:port) did clients in the TinselStream network visit most?

    Click "field statistics" and add "event.url" and "event.port"
    
    *pagead2.googlesyndication.com:443*

* Question 11: Using the 'WindowsEvent' event_source, how many unique Channels is the SIEM receiving Windows event logs from?

    I don't really know what this means, but there's 5 distinct event.Correlation_ActivityID s. Or 5 "event.channel"s if you don't filter on WindowsEvent
    *5*

* Question 12: What is the name of the event.Channel (or Channel) with the second highest number of events?

    Microsoft-Windows-Sysmon/Operational

* Question 13: Our environment is using Sysmon to track many different events on Windows systems. What is the Sysmon Event ID related to loading of a driver?

    We solve this by googling: *6*
    
* Question 14: What is the Windows event ID that is recorded when a new service is installed on a system?

    Once again, this is a question for Google.

    *4697*

* Question 15: Using the WindowsEvent event_source as your initial filter, how many user accounts were created?

    We find this by using the following search: `event_source : "WindowsEvent" and event.EventID : 4720`
    (after googling for the event id).
    
    *0*

### Hard

* Question 1: What is the event.EventID number for Sysmon event logs relating to process creation?
    
    We use Google to find it is *1*
    
* Question 2: How many unique values are there for the 'event_source' field in all of the logs?

    *5* (See easy mode)

* Question 3: What is the event_source name that contains the email logs?

    *SnowGlowMailPxy* (see easy mode)

* Question 4: The North Pole network was compromised recently through a sophisticated phishing attack sent to one of our elves. The attacker found a way to bypass the middleware that prevented phishing emails from getting to North Pole elves. As a result, one of the Received IPs will likely be different from what most email logs contain. Find the email log in question and submit the value in the event 'From:' field for this email log event.

    Looking at the event.ReceivedIP1, all are from a single IP. event.ReceivedIP2 has two IPs; almost
    all are from one (which means its probably legit, unless the attacker sent *lots* of emails and 
    the elves all use TikTok or something) so filter on the second:

    `event_source : "SnowGlowMailPxy" and event.ReceivedIP2 : "34.30.110.62"`

    *kriskring1e@northpole.local*

* Question 5: Our ElfSOC analysts need your help identifying the hostname of the domain computer that established a connection to the attacker after receiving the phishing email from the previous question. You can take a look at our GreenCoat proxy logs as an event source. Since it is a domain computer, we only need the hostname, not the fully qualified domain name (FQDN) of the system.

    In the body, we can see the link provided is http://hollyhaven.snowflake/howtosavexmas.zip so we can
    use this to search:

    `event_source : "GreenCoat" and event.url :  "http://hollyhaven.snowflake/howtosavexmas.zip"`

    *SleighRider*

* Question 6: What was the IP address of the system you found in the previous question?

    *172.24.25.12*

* Question 7: A process was launched when the user executed the program AFTER they downloaded it. What was that Process ID number (digits only please)?

    !!! warning "time filter"
        For the rest of the questions, until otherwise noted, we will filter to times after the
        event used for question 6 and 7, and order by time.

    `event_source : "WindowsEvent" and event.EventID : "1" and (event.Hostname : "SleighRider.northpole.local" or event.Hostname : "SleighRider")`

    *10014*

* Question 8: Did the attacker's payload make an outbound network connection? Our ElfSOC analysts need your help identifying the destination TCP port of this connection.

    `event.SourceIp : "172.24.25.12"`. Note that the oldest log has event.Image
        C:\Users\elf_user02\Downloads\howtosavexmas\howtosavexmas.pdf.exe so it's the log we want.

    *8443*

* Question 9: The attacker escalated their privileges to the SYSTEM account by creating an inter-process communication (IPC) channel. Submit the alpha-numeric name for the IPC channel used by the attacker.

    `event.ProcessID : "10014"` shows the process setting REGISTRY KEYS etc before running `cmd.exe /c echo ddpvccdbr &gt; \\\\.\\pipe\\ddpvccdbr` (on the 34th action). So the answer is *ddpvccdbr*

* Question 10: The attacker's process attempted to access a file. Submit the full and complete file path accessed by the attacker's process.

    `event.Hostname : "SleighRider.northpole.local" and event.EventID : "4663" and event.ProcessName : "C:\Users\elf_user02\Downloads\howtosavexmas\howtosavexmas.pdf.exe"`

    C:\Users\elf_user02\Desktop\kkringl315@10.12.25.24.pem

* Question 11: The attacker attempted to use a secure protocol to connect to a remote system. What is the hostname of the target server?
    
    The previous action seemed to be saving an SSH key, so searching for "ssh" finds messages to
    *kringleSSleigH*

* Question 12: The attacker created an account to establish their persistence on the Linux host. What is the name of the new account created by the attacker?

    Searching for logs around that time with `event.hostname : "kringleSSleigH"` finds an event: `kkringl315 : TTY=pts/5 ; PWD=/opt ; USER=root ; COMMAND=/usr/sbin/adduser ssdh`

    *ssdh*

* Question 13: The attacker wanted to maintain persistence on the Linux host they gained access to and executed multiple binaries to achieve their goal. What was the full CLI syntax of the binary the attacker executed after they created the new user account?

    Filtering for the hostname after the user was added and sorting by time, we see the attacker setting up the user's password, etc, and then they run *kkringl315 : TTY=pts/5 ; PWD=/opt ; USER=root ; COMMAND=/usr/sbin/usermod -a -G sudo ssdh. /usr/sbin/usermod -a -G sudo ssdh*

* Question 14: The attacker enumerated Active Directory using a well known tool to map our Active Directory domain over LDAP. Submit the full ISO8601 compliant timestamp when the first request of the data collection attack sequence was initially recorded against the domain controller.
    
    Search for ldap bind event: `event.EventID : "2889"` and then use the event.date field
    *2024-09-16T11:10:12-04:00*

* Question 15: The attacker attempted to perform an ADCS ESC1 attack, but certificate services denied their certificate request. Submit the name of the software responsible for preventing this initial attack.

    Search for certificate denied event: `event.EventID : "4888"`

    *KringleGuard*

* Question 16: We think the attacker successfully performed an ADCS ESC1 attack. Can you find the name of the user they successfully requested a certificate on behalf of?

    Searching `event.EventID : "4886"`  shows the name *nutcrakr*

* Question 17: One of our file shares was accessed by the attacker using the elevated user account (from the ADCS attack). Submit the folder name of the share they accessed.

    We can search with `event.EventID : "5140" and  "nutcrakr"` to find the file share is

    *WishLists*

* Question 18: The naughty attacker continued to use their privileged account to execute a PowerShell script to gain domain administrative privileges. What is the password for the account the attacker used in their attack payload?

    `event.EventID : "4104" and event.Computer : "SleighRider.northpole.local"`

    *fR0s3nF1@k3_s*

* Question 19: The attacker then used remote desktop to remotely access one of our domain computers. What is the full ISO8601 compliant UTC EventTime when they established this connection?

    For this, we need [the logon type for RDP](https://learn.microsoft.com/en-us/windows-server/identity/securing-privileged-access/reference-tools-logon-types). We can then use it to search:
    
    `event.LogonType : "10" and event.Hostname : "dc01.northpole.local"`

    *2024-09-16T15:35:57.000Z*

* Question 20: The attacker is trying to create their own naughty and nice list! What is the full file path they created using their remote desktop connection?

    There are no signs of file creations or files transferred if we use eventID. Instead, we have to 
    look through the processes the attacker ran with `event.EventID : "1" and event.Hostname : "dc01.northpole.local" and event.User : "NORTHPOLE\\nutcrakr"`

    This shows the attacker creating a new file with Notepad: `"C:\Windows\system32\NOTEPAD.EXE" C:\WishLists\santadms_only\its_my_fakelst.txt`

    *C:\WishLists\santadms_only\its_my_fakelst.txt*

* Question 21: The Wombley faction has user accounts in our environment. How many unique Wombley faction users sent an email message within the domain?

    !!! warning "remove the time filter"

    There's probably a smarter way to do this, but I'm only used to Splunk, so I just used the visualisation of `event_source : "SnowGlowMailPxy"` and counted the wombley like email addresses.
    There are *4*

* Question 22: The Alabaster faction also has some user accounts in our environment. How many emails were sent by the Alabaster users to the Wombley faction users?

    We can use the following query:
    `event_source : "SnowGlowMailPxy" and (event.To : "wcub101@northpole.local" or event.To : "wcub303@northpole.local" or event.To : "wcub808@northpole.local" or event.To : "wcube311@northpole.local" ) and (event.From : "asnowball04@northpole.local" or event.From : "asnowball08@northpole.local" or event.From : "asnowball09@northpole.local" or event.From : "asnowball_05@northpole.local" )` to see there are *22*
    
* Question 23: Of all the reindeer, there are only nine. What's the full domain for the one whose nose does glow and shine? To help you narrow your search, search the events in the 'SnowGlowMailPxy' event source.

    I switched to the Lucene search syntax for this since it supports fuzzy search. We're clearly looking
    for something Rudolph-y, but:
    `event_source:"SnowGlowMailPxy" AND event.From:*Rudolph*` shows 17 results. Only one of the "from"s
    mentions glow, so the domain must be *rud01ph.glow*

* Question 24: With a fiery tail seen once in great years, what's the domain for the reindeer who flies without fears? To help you narrow your search, search the events in the 'SnowGlowMailPxy' event source.

    "Fiery tail seen once in great years" clearly refers to the reindeer Comet, but filtering for that
    didn't seem to work (due to the use of numbers instead of letters). Instead, I took the slow approach of filtering out the emails it could not
    be for with:

    `event_source:"SnowGlowMailPxy" AND (NOT event.From:*elf*) AND (NOT event.From:*northpole.local)`
    and then just looked through the remaining emails to see it must refer to *c0m3t.halleys*


## Response

!!! quote "Fitzy Shortstack"
    Fantastic job! You worked through the logs using the ELK stack like a pro—efficient, quick, and spot-on. Maybe, just maybe, this will turn Santa's frown upside down!

    Up for the real challenge? Take a deep dive into those logs and query your way through the chaos. It might be tricky, but I know your adaptable skills will crack it!

    Unbelievable! You dissected the attack chain using advanced analysis—impressive work! With determination like yours, we might just fix the mess and get Santa smiling again.

    Bravo! You pieced it all together, uncovering the attack path. Santa's gonna be grateful for your quick thinking and tech savvyness. The North Pole owes you big time!
