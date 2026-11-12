# Detecting RDP Brute Force Attacks

## Introduction

We often encounter Remote Desktop Protocol (RDP) brute force attacks as a favorite vector for attackers to gain initial foothold in a network. The concept of an RDP brute force attack is relatively straightforward: attackers attempt to login into a Remote Desktop session by systematically guessing and trying different passwords until they find the correct one. This method exploits the fact that many users often have weak or default passwords that are easy to guess.

## How RDP Traffic Looks Like

<img width="698" height="403" alt="image" src="https://github.com/user-attachments/assets/99a94f39-cecb-4b3e-8154-1a9c1165d281" />

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/rdp_bruteforce`
- **Related Splunk Index:** `rdp_bruteforce`
- **Related Splunk Sourcetype:** `bro:rdp:json`

## Detecting RDP Brute Force Attacks With Splunk & Zeek Logs

Now let's explore how we can identify RDP brute force attacks, using Splunk and Zeek logs.

```shellsession
index="rdp_bruteforce" sourcetype="bro:rdp:json"
| bin _time span=5m
| stats count values(cookie) by _time, id.orig_h, id.resp_h
| where count>30
```

<img width="1318" height="295" alt="image" src="https://github.com/user-attachments/assets/3b20bcbf-6bed-4800-b17c-cd36e2cc07bb" />

The knowledge of how to detect brute-force attacks helps build detection logic and alerts. Brute-force attacks focus on finding services that accept credentials, where attackers may attempt to use a default username in combination with a default password, or they might perform reconnaissance through passive information gathering by observing the format of emails as an initial vector for finding a valid username format. Within this section, we focus on detecting brute-force attacks against RDP, which is one of the protocols that threat actors will attack. In the exercise, we will be placed in a position to detect SSH brute-forcing, which may be implemented across Windows hosts, building upon the methods of understanding what services run in our organization and by developing the process of adaptation in various scenarios.

## 🔑 Key Takeaway

RDP brute force attacks involve repeatedly attempting to log in with different passwords. Splunk and Zeek logs can help detect these attempts by aggregating RDP connection events into five-minute intervals and identifying source and destination pairs with more than 30 events.
