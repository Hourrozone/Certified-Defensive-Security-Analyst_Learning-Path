# Detecting Exfiltration (DNS)

## How DNS Exfiltration Works

Attackers employ DNS-based exfiltration due to its reliability, stealthiness, and the fact that DNS traffic is often allowed by default in network firewall rules. By embedding data within DNS queries and responses, attackers can bypass security controls and exfiltrate data covertly. Below is a detailed explanation of this technique and detection methods:

1. **Initial Compromise:** The attacker gains access to the victim's network, typically through malware, phishing, or exploiting vulnerabilities.
2. **Data Identification and Preparation:** The attacker locates the data they want to exfiltrate and prepares it for transmission. This usually involves encoding or encrypting the data and splitting it into small chunks.
3. **Exfiltration via DNS:** The attacker sends the data in the subdomains of DNS queries, utilizing techniques such as DNS tunneling or fast flux. They typically use a domain under their control or a compromised domain for this purpose. The attacker's DNS server receives the queries, extracts the data, and reassembles it.
4. **Data Retrieval and Analysis:** After exfiltration, the attacker decodes or decrypts the data and analyzes it.

## How DNS Exfiltration Traffic Looks Like

<img width="958" height="247" alt="image" src="https://github.com/user-attachments/assets/1af4f1be-061c-4209-b1e6-d4ff24ba2a6b" />

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/dns_exf`
- **Related Splunk Index:** `dns_exf`
- **Related Splunk Sourcetype:** `bro:dns:json`

## Detecting DNS Exfiltration With Splunk & Zeek Logs

Now let's explore how we can identify DNS exfiltration, using Splunk and Zeek logs.

```shellsession
index=dns_exf sourcetype="bro:dns:json"
| eval len_query=len(query)
| search len_query>=40 AND query!="*.ip6.arpa*" AND query!="*amazonaws.com*" AND query!="*._googlecast.*" AND query!="_ldap.*"
| bin _time span=24h
| stats count(query) as req_by_day by _time, id.orig_h, id.resp_h
| where req_by_day>60
| table _time, id.orig_h, id.resp_h, req_by_day
```

<img width="1329" height="387" alt="image" src="https://github.com/user-attachments/assets/ce1b6ace-5011-4e97-9a2f-0be57c25daae" />

## 🔑 Key Takeaway

DNS exfiltration hides stolen data within DNS queries, often using encoded or encrypted subdomains. By analyzing Zeek DNS logs in Splunk, filtering for unusually long queries, excluding common legitimate DNS patterns, and identifying hosts generating high volumes of DNS requests, analysts can detect potential DNS tunneling and data exfiltration.
