# Detecting Exfiltration (HTTP)

## Introduction

Data exfiltration inside the POST body is a technique that attackers employ to extract sensitive information from a compromised system by disguising it as legitimate web traffic. It involves transmitting the stolen data from the compromised system to an external server controlled by the attacker using HTTP POST requests. Since POST requests are commonly used for legitimate purposes, such as form submissions and file uploads, this method of data exfiltration can be difficult to detect.

To exfiltrate the data, the attackers send it as the body of an HTTP POST request to their command and control (C2) server. They often use seemingly innocuous URLs and headers to further disguise the malicious traffic. The C2 server receives the POST request, extracts the data from the body, and decodes or decrypts it for further analysis and exploitation.

To detect data exfiltration via POST body, we can employ network monitoring and analysis tools to aggregate all data sent to specific IP addresses and ports. By analyzing the aggregated data, we can identify patterns and anomalies that may indicate data exfiltration attempts.

In this section, we will monitor the volume of outgoing traffic from our network to specific IP addresses and ports. If we observe unusually large or frequent data transfers to a specific destination, it may indicate data exfiltration.

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/cobaltstrike_exfiltration_http`
- **Related Splunk Index:** `cobaltstrike_exfiltration_http`
- **Related Splunk Sourcetype:** `bro:http:json`

## Detecting HTTP Exfiltration With Splunk & Zeek Logs

Now let's explore how we can identify HTTP exfiltration, using Splunk and Zeek logs.

```shellsession id="r5v8mc"
index="cobaltstrike_exfiltration_http" sourcetype="bro:http:json" method=POST
| stats sum(request_body_len) as TotalBytes by src, dest, dest_port
| eval TotalBytes = TotalBytes/1024/1024
```

<img width="1331" height="315" alt="image" src="https://github.com/user-attachments/assets/f50d3c6f-ec32-49af-9b85-b27de39080ed" />

## 🔑 Key Takeaway

HTTP POST requests can be abused to exfiltrate sensitive data by hiding it in the request body and disguising the traffic as legitimate web activity. The Splunk query aggregates POST request body sizes by source IP, destination IP, and destination port, then converts the total transferred data from bytes to megabytes to help identify unusually large outbound transfers.
