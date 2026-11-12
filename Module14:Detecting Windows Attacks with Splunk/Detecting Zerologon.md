# Detecting Zerologon

## Introduction

The Zerologon vulnerability, also known as CVE-2020-1472, is a critical flaw in the implementation of the Netlogon Remote Protocol, specifically in the cryptographic algorithm used by the protocol. The vulnerability can be exploited by an attacker to impersonate any computer, including the domain controller, and execute remote procedure calls on their behalf. Let's dive into the technical details of this flaw.

At the heart of Zerologon is the cryptographic issue in the way Microsoft's Netlogon Remote Protocol authenticates users and machines in a Windows domain. When a client wants to authenticate against the domain controller, it uses a protocol called MS-NRPC, a part of Netlogon, to establish a secure channel.

During this process, the client and the server generate a session key, which is computed from the machine account's password. This key is then used to derive an initialization vector (IV) for the AES-CFB8 encryption mode. In a secure configuration, the IV should be unique and random for each encryption operation. However, due to the flawed implementation in the Netlogon protocol, the IV is set to a fixed value of all zeros.

The attacker can exploit this cryptographic weakness by attempting to authenticate against the domain controller using a session key consisting of all zeros, effectively bypassing the authentication process. This allows the attacker to establish a secure channel with the domain controller without knowing the machine account's password.

Once this channel is established, the attacker can utilize the `NetrServerPasswordSet2` function to change the computer account's password to any value, including a blank password. This effectively gives the attacker full control over the domain controller and, by extension, the entire Active Directory domain.

The Zerologon vulnerability is particularly dangerous due to its simplicity and the level of access it provides to attackers. Exploiting this flaw requires only a few Netlogon messages, and it can be executed within seconds.

## How Zerologon Looks Like From A Network Perspective

<img width="817" height="722" alt="image" src="https://github.com/user-attachments/assets/2c87a20a-f8e6-45e0-96bb-1e05d064c820" />

**Image Source:** https://www.trendmicro.com/en_us/what-is/zerologon.html

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/zerologon`
- **Related Splunk Index:** `zerologon`
- **Related Splunk Sourcetype:** `bro:dce_rpc:json`

## Detecting Zerologon With Splunk & Zeek Logs

Now let's explore how we can identify Zerologon, using Splunk and Zeek logs.

```shellsession id="z6n2kp"
index="zerologon" endpoint="netlogon" sourcetype="bro:dce_rpc:json"
| bin _time span=1m
| where operation == "NetrServerReqChallenge" OR operation == "NetrServerAuthenticate3" OR operation == "NetrServerPasswordSet2"
| stats count values(operation) as operation_values dc(operation) as unique_operations by _time, id.orig_h, id.resp_h
| where unique_operations >= 2 AND count>100
```

<img width="1333" height="411" alt="image" src="https://github.com/user-attachments/assets/771f94fb-b760-4242-81d2-641a0e16de9b" />

## 🔑 Key Takeaway

Zerologon (CVE-2020-1472) exploits a cryptographic weakness in the Netlogon Remote Protocol that allows attackers to bypass authentication and potentially take control of a domain controller. The Splunk query identifies potential exploitation by monitoring Netlogon RPC operations and flagging source-destination pairs that execute at least two distinct operations more than 100 times within a one-minute interval.
