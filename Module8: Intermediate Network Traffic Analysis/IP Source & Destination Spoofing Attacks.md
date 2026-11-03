# IP Source & Destination Spoofing Attacks

There are many cases where we might see irregular traffic for IPv4 and IPv6 packets. In many such cases, this might be done through the source and destination IP fields. We should always consider the following when analyzing these fields for our traffic analysis efforts.

- **The Source IP Address should always be from our subnet** - If we notice that an incoming packet has an IP source from outside of our local area network, this can be an indicator of packet crafting.
- **The Source IP for outgoing traffic should always be from our subnet** - If the source IP is from a different IP range than our own local area network, this can be an indicator of malicious traffic that is originating from inside our network.

An attacker might conduct these packet crafting attacks towards the source and destination IP addresses for many different reasons or desired outcomes. Here are a few that we can look for:

- **Decoy Scanning** - In an attempt to bypass firewall restrictions, an attacker might change the source IP of packets to enumerate further information about a host in another network segment. Through changing the source to something within the same subnet as the target host, the attacker might succeed in firewall evasion.
- **Random Source Attack DDoS** - Through random source crafting an attacker might be able to send tons of traffic to the same port on the victim host. This in many cases, is used to exhaust resources of our network controls or on the destination host.
- **LAND Attacks** - LAND Attacks operate similarly to Random Source denial-of-service attacks in the nature that the source address is set to the same as the destination hosts. In doing so the attacker might be able to exhaust network resources or cause crashes on the target host.
- **SMURF Attacks** - Similar to LAND and Random Source attacks, SMURF attacks work through the attacker sending large amounts of ICMP packets to many different hosts. However, in this case the source address is set to the victim machines, and all of the hosts which receive this ICMP packet respond with an ICMP reply causing resource exhaustion on the crafted source address (victim).
- **Initialization Vector Generation** - In older wireless networks such as wired equivalent privacy, an attacker might capture, decrypt, craft, and re-inject a packet with a modified source and destination IP address in order to generate initialization vectors to build a decryption table for a statistical attack. These can be seen in nature by noticing an excessive amount of repeated packets between hosts.

It is important to note, that unlike ARP poisoning, the attacks we will be exploring in this section derive from IP layer communications and not ARP poisoning necessarily. However, these attacks tend to be conducted in tandem for most nefarious activities.

## Finding Decoy Scanning Attempts

**Related PCAP File(s):**

- `decoy_scanning_nmap.pcapng`

Simply put, when an attacker wants to gather information, they might change their source address to be the same as another legitimate host, or in some cases entirely different from any real host. This is to attempt to evade IDS/Firewall controls, and it can be easily observed.

In the case of decoy scanning, we will notice some strange behavior.

- Initial Fragmentation from a fake address
- Some TCP traffic from the legitimate source address

<img width="1144" height="445" alt="image" src="https://github.com/user-attachments/assets/4285e0de-3990-4191-b8c1-762d4bd1e58a" />

Secondarily, in this attack the attacker might be attempting to cloak their address with a decoy, but the responses for multiple closed ports will still be directed towards them with the RST flags denoted for TCP.

<img width="1137" height="172" alt="image" src="https://github.com/user-attachments/assets/f327d3a9-f813-4408-bd6c-d95561b33c16" />

We will definitely notice this in the case of a large port block which has no services running on the victim host.

<img width="1142" height="374" alt="image" src="https://github.com/user-attachments/assets/d37d53e4-3377-4715-9e86-9b3f512067d0" />

As such, another simple way that we can prevent this attack beyond just detecting it through our traffic analysis efforts is the following.

- **Have our IDS/IPS/Firewall act as the destination host would** - In the sense that reconstructing the packets gives a clear indication of malicious activity.
- **Watch for connections started by one host, and taken over by another** - The attacker after all has to reveal their true source address in order to see that a port is open. This is strange behavior and we can define our rules to prevent it.

## Finding Random Source Attacks

**Related PCAP File(s):**

- `ICMP_rand_source.pcapng`
- `ICMP_rand_source_larg_data.pcapng`
- `TCP_rand_source_attacks.pcapng`

On the opposite side of things, we can begin to explore denial-of-service attacks through source and destination address spoofing. One of the primary and notable examples is random source attacks. These can be conducted in many different flavors. However, notably this can be done like the opposite of a SMURF attack, in which many hosts will ping one host which does not exist, and the pinged host will ping back all others and get no reply.

<img width="990" height="239" alt="image" src="https://github.com/user-attachments/assets/1cfa0c08-f2d4-4c22-9532-9557908d02e1" />

We should also consider that attackers might fragment these random hosts communications in order to draw out more resource exhaustion.

<img width="1179" height="361" alt="image" src="https://github.com/user-attachments/assets/a964c825-70b8-434c-aae3-c6a46522ca01" />

However in many cases, like LAND attacks, these attacks will be used by attackers to exhaust resources to one specific service on a port. Instead of spoofing the source address to be the same as the destination, the attacker might randomize them. We might notice the following.

<img width="872" height="310" alt="image" src="https://github.com/user-attachments/assets/8f4043df-9a9b-4c27-b68a-8c2b9c23a90f" />

In this case, we have a few indicators of nefarious behavior:

- Single Port Utilization from random hosts
- Incremental Base Port with a lack of randomization
- Identical Length Fields

In many real world cases, like a web server, we may have many different users utilizing the same port. However, these requests are contrary of our indicators. Such that they will have different lengths and the base ports will not exhibit this behavior.

## Finding Smurf Attacks

SMURF Attacks are a notable distributed denial-of-service attack, in the nature that they operate through causing random hosts to ping the victim host back. Simply put, an attacker conducts these like the following:

1. The attacker will send an ICMP request to live hosts with a spoofed address of the victim host
2. The live hosts will respond to the legitimate victim host with an ICMP reply
3. This may cause resource exhaustion on the victim host

One of the things we can look for in our traffic behavior is an excessive amount of ICMP replies from a single host to our affected host. Sometimes attackers will include fragmentation and data on these ICMP requests to make the traffic volume larger.

<img width="1171" height="360" alt="image" src="https://github.com/user-attachments/assets/6a65bc6d-d10c-4166-947e-9207b9720246" />

We might notice many different hosts pinging our single host, and in this case it represents the basic nature of SMURF attacks.

<img width="1107" height="462" alt="image" src="https://github.com/user-attachments/assets/6544a90e-7465-4b49-a461-98a59e7c94dc" />

**Image From:** https://techofide.com/blogs/what-is-smurf-attack-what-is-the-denial-of-service-attack-practical-ddos-attack-step-by-step-guide/

## Finding LAND Attacks

**Related PCAP File(s):**

- `LAND-DoS.pcapng`

LAND attacks operate through an attacker spoofing the source IP address to be the same as the destination. These denial-of-service attacks work through sheer volume of traffic and port re-use. Essentially, if all base ports are occupied, it makes real connections much more difficult to establish to our affected host.

<img width="875" height="328" alt="image" src="https://github.com/user-attachments/assets/9e9ccf43-2f4b-4171-bafe-620684964302" />

## 🔑 Key Takeaway

IP source and destination spoofing attacks manipulate packet addressing to evade security controls, conceal scanning activity, or overwhelm network resources. Decoy scanning, random source attacks, SMURF attacks, and LAND attacks each exhibit distinct traffic patterns. During analysis, look for unexpected source IP addresses, fragmented packets, unusual port usage, identical packet lengths, excessive ICMP replies, and connections that appear to switch between different source hosts. These indicators can help identify spoofing activity and support effective IDS/IPS and firewall defenses.
