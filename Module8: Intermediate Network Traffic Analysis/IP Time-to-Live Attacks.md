<img width="872" height="258" alt="image" src="https://github.com/user-attachments/assets/e261f2c0-ae3e-41cd-aaa7-6cecab5b0ceb" /># IP Time-to-Live Attacks

**Related PCAP File(s):**

- `ip_ttl.pcapng`

Time-to-Live attacks are primarily utilized as a means of evasion by attackers. Basically speaking the attacker will intentionally set a very low TTL on their IP packets in order to attempt to evade firewalls, IDS, and IPS systems. These work like the following.

<img width="700" height="365" alt="image" src="https://github.com/user-attachments/assets/1b03ef7f-03fd-4f5f-92c6-f81a32c45fbd" />

1. The attacker will craft an IP packet with an intentionally low TTL value (1, 2, 3 and so on).
2. Through each host that this packet passes through this TTL value will be decremented by one until it reaches zero.
3. Upon reaching zero this packet will be discarded. The attacker will try to get this packet discarded before it reaches a firewall or filtering system to avoid detection/controls.
4. When the packets expire, the routers along the path generate ICMP Time Exceeded messages and send them back to the source IP address.

## Finding Irregularities in IP TTL

For starters, we can begin to dump our traffic and open it in Wireshark. Detecting this in small amounts can be difficult, but fortunately for us attackers will most times utilize ttl manipulation in port scanning efforts. Right away we might notice something like the following.

<img width="872" height="258" alt="image" src="https://github.com/user-attachments/assets/b22b10e8-850b-4698-bff6-a786ae977edf" />

However, we might also notice a returned SYN, ACK message from one of our legitimate service ports on our affected host. In doing so, the attacker might have successfully evaded one of our firewall controls.

<img width="1027" height="460" alt="image" src="https://github.com/user-attachments/assets/e74f2b1e-3fc2-4808-b33d-2a37f28f43d2" />

So, if we were to open one of these packets, we could realistically see why this is. Suppose we opened the IPv4 tab in Wireshark for any of these packets. We might notice a very low TTL like the following.

<img width="500" height="255" alt="image" src="https://github.com/user-attachments/assets/b06963fe-73c4-4b37-ad44-db03d86b12e7" />

As such, we can implement a control which discards or filters packets that do not have a high enough TTL. In doing so, we can prevent these forms of IP packet crafting attacks.

## 🔑 Key Takeaway

IP Time-to-Live attacks attempt to evade firewalls, IDS, and IPS systems by crafting packets with unusually low TTL values. Monitoring for irregular TTL values, especially during port scans, can help identify these attempts. Filtering or discarding packets with insufficient TTL values can help prevent this form of evasion.
