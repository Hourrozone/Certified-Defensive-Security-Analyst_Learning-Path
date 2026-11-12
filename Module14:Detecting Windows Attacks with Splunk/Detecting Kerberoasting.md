# Detecting Kerberoasting

## Introduction

In 2016, a number of blog posts and articles emerged discussing the tactic of querying Service Principal Name (SPN) accounts and their corresponding tickets, an attack that came to be known as Kerberoasting. By possessing just one legitimate user account and its password, an attacker could retrieve the SPN tickets and attempt to break them offline.

After examining numerous resources on kerberoasting, it is evident that RC4 is utilized for ticket encryption behind the scenes. We will exploit this underpinning as a detection point in this section.

**Evidence Source:** https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/t1208-kerberoasting

## How Kerberoasting Traffic Looks Like

<img width="925" height="388" alt="image" src="https://github.com/user-attachments/assets/ef1b5e57-99cf-45be-b2ec-14b880b5f429" />

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/sharphound`
- **Related Splunk Index:** `sharphound`
- **Related Splunk Sourcetype:** `bro:kerberos:json`

## Detecting Kerberoasting With Splunk & Zeek Logs

Now let's explore how we can identify Kerberoasting, using Splunk and Zeek logs.

```shellsession id="w7m3kp"
index="sharphound" sourcetype="bro:kerberos:json"
request_type=TGS cipher="rc4-hmac" 
forwardable="true" renewable="true"
| table _time, id.orig_h, id.resp_h, request_type, cipher, forwardable, renewable, client, service
```

<img width="1901" height="811" alt="image" src="https://github.com/user-attachments/assets/dd600e67-36d7-42d3-8571-a13a285d8cd0" />

## 🔑 Key Takeaway

Kerberoasting allows an attacker with a legitimate domain account to request Service Principal Name (SPN) tickets and attempt to crack them offline. The Splunk query looks for TGS requests encrypted with RC4-HMAC and filters for tickets marked as forwardable and renewable, helping identify potential Kerberoasting activity.
