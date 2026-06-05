---
title: Mischarachterized MicroTik Attack - What Stuck In The Honey
description: Telegram, crypto, and... MikroTik routers?
categories: [honeypot analysis]
tags: [dshield, SANS ISC]
---

My DShield honeypot observed a very tight attack interested in 3 dissimilar vectors:
    • Messaging capabilities (Telegram and SMS)
    • Cryptocurrency mining
    • MikroTik Router check

This attack has been observed in the past and analyzed, with the consensus being that this attack is a botnet attack against MikroTik devices. I believe this is a mischaracterization of this attack.

# Initial Access
This attack, like many others on the internet, attempts to gain access by abusing guessable credentials over SSH. This attack was observed to use:
root:12345
root:admin
root:guest
root:root

# Content

![Alt Text](/assets/img/00001.png)

What first struck me about this attack is how tight it is. Every IP which executed this attack only executed this attack a single time, then never interacted with this honeypot again. This suggests that these hosts are not used for general internet scanning, but rather targeted scanning (or even just this one attack, but that would require much more data to claim).

Also, the key commands executed during this attack were only executed by these IPs, so they are specific to this attack.

About half of the IPs originate from telecoms, where the other half originate from cloud providers. While this does suggest the use of previously compromised devices to run this attack, I don’t believe this constitutes a botnet, at least in the typical usage of the term. The typical usage of a botnet includes using the combined efforts of many/all bots in the network in a single effort. In contrast, here we see a very small set of hosts each running this attack individually, not collectively (see the timeline below). Overall, I believe this suggests that this attack is being preformed by one attacker using a small set of compromised hosts.

# Timing
![Alt Text](/assets/img/00002.png)

In addition to the content of the attack, the timing is also very tight. This honeypot observed this attack 10 times from April 29th to May 13th, 2026. This attack was observed roughly every 1-2 days, with a distinct gap between May 5th and May 10th. This timing leads me to believe that only one actor is preforming this attack, otherwise we would likely see less consistent timing as individual attackers run their scans, as opposed to what we see appears to be a methodical, periodic scan.
Targets
What struck me next was the rather disjointed set of services this attack is targeting. In additional to the usual reconnaissance commands to determine OS, CPU architecture, platform, etc, there were 3 unique commands targeting 3 very distinct vectors:

## Messaging Capabilities
Commands:
```
ls -la ~/.local/share/TelegramDesktop/tdata /home/*/.local/share/TelegramDesktop/tdata /dev/ttyGSM* /dev/ttyUSB-mod* /var/spool/sms/* /var/log/smsd.log /etc/smsd.conf* /usr/bin/qmuxd /var/qmux_connect_socket /etc/config/simman /dev/modem* /var/config/sms/*
```
```
locate D877F783D5D3EF8Cs
```

The first command enumerates on directories associated with Telegram and SMS messaging. The second commands looks for the file ‘D877F783D5D3EF8Cs’, which is used by Telegram to store
“… user ID and encryption key used for interaction between the desktop client and Telegram servers.” (https://malware.news/t/cloud-atlas-seen-using-a-new-tool-in-its-attacks/89672).

I believe the intent is to use any found messaging capabilities to send malicious or otherwise unwanted texts.

## Cryptocurrency Mining
Command:
```
ps | grep '[Mm]iner'
```

This simple command checks to see if any running processes contain the phrase ‘miner’, indicative of cryptocurrency mining software. I believe the intent here is likely to:
    1. Install a crypto miner is one is not already present
    2. If a miner is already present, silently switch out the wallet which is associated with the mined crypto to one controlled by the threat actor, thus sending the threat actor all the crypto mined on that system (Side note: This type of attack reminds me of the Payroll Pirate tactic - https://www.microsoft.com/en-us/security/blog/2025/10/09/investigating-targeted-payroll-pirate-attacks-affecting-us-universities/)

## And… MikroTik Routers?
Command:
```
/ip cloud print
```

This command attempts to get the IP and DNS name of a MikroTik RouterOS device. Note that this command still exists on recent versions of RouterOS (I tested on RouterOS 7.19.1).

Sample output - https://help.mikrotik.com/docs/spaces/ROS/pages/97779929/Cloud

```
[admin@MikroTik] /ip cloud print
         ddns-enabled: yes
 ddns-update-interval: none
          update-time: yes
       public-address: 159.148.147.196
  public-address-ipv6: 2a02:610:7501:1000::2
             dns-name: 529c0491d41c.sn.mynetname.net
               status: updated
```

# Mischaracterized?
This attack is by no means new. A very similar variant (just missing the Telegram piece) has been detailed at least a few times now (https://exylum.tech/blog/honeypot-24-02.html, https://malwaremily.medium.com/honeypot-logs-a-botnets-search-for-mikrotik-routers-48e69e110e52). But I believe this attack is being mischaracterized as targeting MikroTik devices.

This attack does run a command specific to MikroTik devices. But it only runs one. All the other commands executed in this attack (ls, cat, ifconfig, echo, uname) do not exist in RouterOS. It is possible that, if the target does happen to be a MicoTik device, the attack changes dynamically to account for that. But given the behavior observed, this attack targets Linux devices.

Additionally, the queries for crypto miners would not make sense to run on a router. Even beefy enterprise routers have very limited compute resources when compared to end-user computing devices, and consuming those resources is much more likely to be noticed when the router starts dropping packets as a result of resource exhaustion.

Similarly, the queries for messaging platforms would not make sense on a MikroTik device. For SMS, while you can connect a SMS modem to a MikroTik Router, and while that connection can be over serial, it is not using SMSD or QMUXD, and does not use the directories which this attack queries for. Additionally, even if those directories did exist, you cannot detect those directories using the ls command, because the ls command does not exist on MikroTik Routers. If this were targeting MikroTik routers, I would expect the command `/tool sms` to be used instead (https://help.mikrotik.com/docs/spaces/ROS/pages/62390313/SMS). And it’s obvious why Telegram would not be present on a MikroTik device, though that piece is not present in the existing analysis.

Rather, I would characterize this attack as a multi-vector Linux focused attack targeting messaging and crypto miners, with a check for if it’s running on a MikroTik device.

# Defenses
To defend against this attack, just employ the fundamentals of cybersecurity:

- Preventative:
    - Vector-oriented Defense in Depth - Do not expose management interfaces to the internet.
    - Strong Authentication - Don’t use guessable credentials. Also, use multi-factor authentication wherever possible.
    - Patching - Keep on top of OS and firmware updates, especially for critical or otherwise higher risk systems.
    - Software inventory + Configuration management - Ensure that any software which is not necessary for a system’s operation remains absent.
- Detective:
    - Logging - Ensure critical systems log all interactions, and that those logs are ingested into an analysis and alerting system.
    - Configuration management - Ensure the configuration of all critical systems is baselined and actively monitored.
- Corrective:
    - Configuration management - Ensure that the known-good configuration of all critical systems is detailed such that they can be completely rebuilt.
- Recovery:
    - High availability - Ensure all critical systems are configured with sufficient high availability measures to reduce the likelihood of this attack causing significant operational impact, as dictated by that asset’s RTO and uptime requirements.

# Limitations
This data was collected from a DShield honeypot. Because this honeypot does not have any of the artifacts this attack is interested in (Telegram files, SMS files and services, crypto miners), this attack has likely been observed at it’s most basic, reconnaissance-only form. This attack may include exploitation and persistence actions if the target is interesting, but DShield is not.
This analysis was preformed on a DShield log corpus which collects inbound traffic from 1 residential public IP, and which spans ~1 month at time of writing. This data set is sufficient for analyzing this attack as a point in time, but not for significant longitudinal analysis. Behavioral and longitudinal analysis (usage of individual IPs, posited number of threat actors, etc) must be understood with this limitation in mind.

# Artifacts
## IPs
59.22.201.143  
116.34.14.135  
212.8.39.73  
92.33.220.174  
152.67.93.207  
39.123.115.235  
93.62.72.229  
219.78.63.235  
46.236.167.19  
58.75.223.46  
121.128.173.237  
47.82.157.99  
49.64.179.135  

## Commands
`/ip cloud print`  
`ifconfig`  
`uname -a`   
`cat /proc/cpuinfo`  
`ps | grep '[Mm]iner'`  
`ps -ef | grep '[Mm]iner'`  
`ls -la ~/.local/share/TelegramDesktop/tdata /home/*/.local/share/TelegramDesktop/tdata /dev/ttyGSM* /dev/ttyUSB-mod* /var/spool/sms/* /var/log/smsd.log /etc/smsd.conf* /usr/bin/qmuxd /var/qmux_connect_socket /etc/config/simman /dev/modem* /var/config/sms/*`  
`locate D877F783D5D3EF8Cs`  
`echo Hi | cat -n`  

# AI Use Disclosure
AI was not used to write any of this analysis. Qwen3 8B was used as one step in a pipeline to enrich the collected DShield logs. The development of the DShield log analysis tool pictured was assisted by AI.
