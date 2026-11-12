# Detecting Kerberos Brute Force Attacks

## Introduction

When adversaries perform Kerberos-based user enumeration, they send an AS-REQ (Authentication Service Request) message to the Key Distribution Center (KDC), which is responsible for handling Kerberos authentication. This message includes the username they're trying to validate. They pay close attention to the response they receive, as it reveals valuable information about the existence of the specified user account.

A valid username will prompt the server to return a TGT or raise an error like `KRB5KDC_ERR_PREAUTH_REQUIRED`, indicating that preauthentication is required. On the other hand, an invalid username will be met with a Kerberos error code `KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN` in the AS-REP (Authentication Service Response) message. By examining the responses to their AS-REQ messages, adversaries can quickly determine which usernames are valid on the target system.

## How Kerberos Brute Force Attacks Look Like On The Wire

<img width="810" height="252" alt="image" src="https://github.com/user-attachments/assets/216d82e8-db16-4491-b434-98aacdc089fb" />

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/kerberos_bruteforce`
- **Related Splunk Index:** `kerberos_bruteforce`
- **Related Splunk Sourcetype:** `bro:kerberos:json`

## Detecting Kerberos Brute Force Attacks With Splunk & Zeek Logs

Now let's explore how we can identify Kerberos brute force attacks, using Splunk and Zeek logs.

```shellsession id="k8j2wp"
index="kerberos_bruteforce" sourcetype="bro:kerberos:json"
error_msg!=KDC_ERR_PREAUTH_REQUIRED
success="false" request_type=AS
| bin _time span=5m
| stats count dc(client) as "Unique users" values(error_msg) as "Error messages" by _time, id.orig_h, id.resp_h
| where count>30
```

<img width="999" height="369" alt="image" src="https://github.com/user-attachments/assets/4221e981-4169-40bc-aea1-ec008e426503" />

## 🔑 Key Takeaway

Kerberos brute force and user enumeration attacks exploit AS-REQ responses to identify valid usernames. Valid accounts may return a TGT or `KRB5KDC_ERR_PREAUTH_REQUIRED`, while invalid accounts return `KRB5KDC_ERR_C_PRINCIPAL_UNKNOWN`. The Splunk query identifies potential attacks by filtering unsuccessful AS requests, grouping events into five-minute intervals, and flagging sources with more than 30 requests.
