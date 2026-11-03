# ARP Scanning & Denial-of-Service

We might discern additional aberrant behaviors within the ARP requests and replies. It is common knowledge that poisoning and spoofing form the core of most ARP-based denial-of-service (DoS) and man-in-the-middle (MITM) attacks. However, adversaries could also exploit ARP for information gathering. Thankfully, we possess the skills to detect and evaluate these tactics following similar procedures.

## ARP Scanning Signs

**Related PCAP File(s):**

- `ARP_Scan.pcapng`

Some typical red flags indicative of ARP scanning are:

- Broadcast ARP requests sent to sequential IP addresses (.1,.2,.3,...)
- Broadcast ARP requests sent to non-existent hosts
- Potentially, an unusual volume of ARP traffic originating from a malicious or compromised host

## Finding ARP Scanning

Without delay, if we were to open the related traffic capture file (ARP_Scan.pcapng) in Wireshark and apply the filter `arp.opcode`, we might observe the following:

<img width="927" height="345" alt="image" src="https://github.com/user-attachments/assets/5a6a9dde-74af-4556-81ee-eeb12e593a30" />

It's possible to detect that indeed ARP requests are being propagated by a single host to all IP addresses in a sequential manner. This pattern is symptomatic of ARP scanning and is a common feature of widely-used scanners such as Nmap.

Furthermore, we may discern that active hosts respond to these requests via their ARP replies. This could signal the successful execution of the information-gathering tactic by the attacker.

## Identifying Denial-of-Service

**Related PCAP File(s):**

- `ARP_Poison.pcapng`

An attacker can exploit ARP scanning to compile a list of live hosts. Upon acquiring this list, the attacker might alter their strategy to deny service to all these machines. Essentially, they will strive to contaminate an entire subnet and manipulate as many ARP caches as possible. This strategy is also plausible for an attacker seeking to establish a man-in-the-middle position.

<img width="1206" height="463" alt="image" src="https://github.com/user-attachments/assets/03243686-ab47-44ef-861e-f0854d8f315b" />

Promptly, we might note that the attacker's ARP traffic may shift its focus towards declaring new physical addresses for all live IP addresses. The intent here is to corrupt the router's ARP cache.

Conversely, we may witness the duplicate allocation of 192.168.10.1 to client devices. This indicates that the attacker is attempting to corrupt the ARP cache of these victim devices with the intention of obstructing traffic in both directions.

<img width="1042" height="256" alt="image" src="https://github.com/user-attachments/assets/5e06ce24-53c4-4126-abfb-cb46745dcd4b" />

## Responding To ARP Attacks

Upon identifying any of these ARP-related anomalies, we might question the suitable course of action to counter these threats. Here are a couple of possibilities:

1. **Tracing and Identification:** First and foremost, the attacker's machine is a physical entity located somewhere. If we manage to locate it, we could potentially halt its activities. On occasions, we might discover that the machine orchestrating the attack is itself compromised and under remote control.
2. **Containment:** To stymie any further exfiltration of information by the attacker, we might contemplate disconnecting or isolating the impacted area at the switch or router level. This action could effectively terminate a DoS or MITM attack at its source.

Link layer attacks often fly under the radar. While they may seem insignificant to identify and investigate, their detection could be pivotal in preventing the exfiltration of data from higher layers of the OSI model.

## 🔑 Key Takeaway

ARP scanning can reveal live hosts by sending sequential broadcast requests across a subnet. Attackers may then use the discovered hosts to launch ARP poisoning attacks, corrupt ARP caches, and cause denial-of-service or man-in-the-middle conditions. Detecting unusual ARP request patterns, duplicate IP-to-MAC mappings, and excessive ARP traffic can help identify these attacks. Responding through tracing, identification, and containment can prevent further compromise and data exfiltration.
