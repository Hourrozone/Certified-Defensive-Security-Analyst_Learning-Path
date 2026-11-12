# Detecting Nmap Port Scanning

## Introduction

Port scanning with Nmap is a key technique in the toolkit of attackers and penetration testers alike. In essence, what we're doing with Nmap is probing networked systems for open ports - these are the 'gates' through which data passes in and out of a system. Open ports can be likened to doors that might be unlocked in a building - doors that attackers could potentially use to gain access.

When we use Nmap for port scanning, we're basically initiating a series of connection requests. We systematically attempt to establish a TCP handshake with each port in the target's address space. If the connection is successful, it indicates that the port is open. This is where it gets interesting. When we connect to an open port, the service listening on that port might send back a "banner" - this is essentially a little bit of data that tells us what service is running, and maybe even what version it's running.

But let's clear up a misconception - when we're talking about Nmap sending data to the scanning port, we're not actually sending any real data. Aside from the actual TCP handshake itself, the payload of the packets Nmap sends is zero. We're not sending any extra data; we're just trying to initiate a connection.

## Related Evidence

- **Related Directory:** `/home/htb-student/module_files/cobaltstrike_beacon`
- **Related Splunk Index:** `cobaltstrike_beacon`
- **Related Splunk Sourcetype:** `bro:conn:json`

## Detecting Nmap Port Scanning With Splunk & Zeek Logs

Now let's explore how we can identify Nmap port scanning, using Splunk and Zeek logs.

```shellsession
index="cobaltstrike_beacon" sourcetype="bro:conn:json" orig_bytes=0 dest_ip IN (192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8) 
| bin span=5m _time 
| stats dc(dest_port) as num_dest_port by _time, src_ip, dest_ip 
| where num_dest_port >= 3
```

<img width="927" height="325" alt="image" src="https://github.com/user-attachments/assets/bfa26026-9bdf-4748-9cff-f1e5df45ac4a" />

## Search Breakdown

- **`index="cobaltstrike_beacon"`:** This restricts the search to logs stored in the cobaltstrike_beacon index.
- **`orig_bytes=0`:** This part of the search filter focuses on network events where the original bytes sent are zero.
- **`dest_ip IN (192.168.0.0/16, 172.16.0.0/12, 10.0.0.0/8)`:** This restricts the search to network events where the destination IP address is within the private IP address ranges, which are commonly used in internal networks.
- **`| bin span=5m _time`:** This command bins the events into 5-minute intervals based on the _time field, which is the timestamp of each event.
- **`| stats dc(dest_port) as num_dest_port by _time, src_ip, dest_ip`:** The stats command is used to aggregate data. The dc(dest_port) function counts the distinct number of destination ports accessed for each combination of _time, src_ip, and dest_ip. The result is stored in a new field called num_dest_port.
- **`| where num_dest_port >= 3`:** This part of the search filters the results to only show those records where the distinct count of destination ports (num_dest_port) is three or greater. This is based on the assumption that scanning three or more ports within a short time frame is a potential indicator of a port scan.

## 🔑 Key Takeaway

Nmap port scanning probes networked systems for open ports by initiating TCP connection requests. The Splunk query identifies potential scans by finding events with zero original bytes sent to private destination IP addresses, then counting distinct destination ports accessed by each source and destination IP within five-minute intervals. Three or more distinct destination ports may indicate a port scan.
