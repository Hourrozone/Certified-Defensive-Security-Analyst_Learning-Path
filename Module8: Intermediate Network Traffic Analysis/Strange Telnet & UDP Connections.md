# Strange Telnet & UDP Connections

When we look for strange traffic, we should always consider telnet and UDP traffic. After all, these can be overlooked, but can especially revealing during our traffic analysis efforts.

## Telnet

<img width="800" height="600" alt="image" src="https://github.com/user-attachments/assets/66c9ea57-9268-47b2-9e8a-bc6585d54f08" />

Telnet is a network protocol that allows a bidirectional interactive communication session between two devices over a network. This protocol was developed in the 1970s and was defined in RFC 854. As of recent years, its usage has decreased significantly as opposed to SSH.

In many older cases, such as our Windows NT like machines, they may still utilize telnet to provide remote command and control to microsoft terminal services.

However, we should always watch for weird and strange telnet communications as it can also be used by attackers for malicious purposes such as data exfiltration and tunneling.

### Finding Traditional Telnet Traffic Port 23

**Related PCAP File(s):**

- `telnet_tunneling_23.pcapng`

Suppose we were to open Wireshark, we might notice some telnet communications originating from Port 23. In this case, we can always inspect this traffic further.

<img width="1391" height="411" alt="image" src="https://github.com/user-attachments/assets/e01392a7-03a1-4331-97af-4467752bafee" />

Fortunately for us, telnet traffic tends to be decrypted and easily inspectable, but like ICMP, DNS, and other tunneling methods, attackers may encrypt, encode, or obfuscate this text. So we should always be careful.

<img width="1019" height="115" alt="image" src="https://github.com/user-attachments/assets/e20e603e-3e91-4bd4-8efd-70b34073ba80" />

### Unrecognized TCP Telnet in Wireshark

**Related PCAP File(s):**

- `telnet_tunneling_9999.pcapng`

Telnet is just a communication protocol, and as such can be easily switched to another port by an attacker. Keeping an eye on these strange port communications can allow us to find potentially malicious actions. Lets take the following for instance.

<img width="1408" height="411" alt="image" src="https://github.com/user-attachments/assets/2102b763-bb3f-4275-8746-a14650fdb00e" />

We may see a ton of communications from one client on port 9999. We can dive into this a little further by looking at the contents of these communications.

<img width="543" height="101" alt="image" src="https://github.com/user-attachments/assets/bc508645-a15e-4ba4-b76d-0370aa987b91" />

If we noticed something like above, we would want to follow this TCP stream.

<img width="472" height="269" alt="image" src="https://github.com/user-attachments/assets/f1f1caf1-4409-4cd9-8913-ca06b63e7d13" />

Doing so can allow us to inspect potentially malicious actions.

### Telnet Protocol through IPv6

**Related PCAP File(s):**

- `telnet_tunneling_ipv6.pcapng`

After all, unless our local network is configured to utilize IPv6, observing IPv6 traffic can be an indicator of bad actions within our environment. We might notice the usage of IPv6 addresses for telnet like the following.

<img width="1172" height="330" alt="image" src="https://github.com/user-attachments/assets/513c7552-d4b7-46c8-8033-bc46f49500d1" />

We can narrow down our filter in Wireshark to only show telnet traffic from these addresses with the following filter.

```text
((ipv6.src_host == fe80::c9c8:ed3:1b10:f10b) or (ipv6.dst_host == fe80::c9c8:ed3:1b10:f10b)) and telnet
```

<img width="734" height="311" alt="image" src="https://github.com/user-attachments/assets/9701fe7e-544a-4ca3-b016-d795966dd9b0" />

Likewise, we can inspect the contents of these packets through their data field, or by following the TCP stream.

<img width="1027" height="107" alt="image" src="https://github.com/user-attachments/assets/290fbc01-56e3-4a0e-8918-eaa9aefce21c" />

## Watching UDP Communications

**Related PCAP File(s):**

- `udp_tunneling.pcapng`

On the other hand, attackers might opt to use UDP connections over TCP in their exfiltration efforts.

<img width="600" height="360" alt="image" src="https://github.com/user-attachments/assets/a140469c-7ca9-4af0-8509-8e15da16f055" />

One of the biggest distinguishing aspects between TCP and UDP is that UDP is connectionless and provides fast transmission. Let's take the following traffic for instance.

<img width="773" height="377" alt="image" src="https://github.com/user-attachments/assets/6c5ca0ef-26e2-46bd-9c62-a11fb16230c5" />

We will notice that instead of a SYN, SYN/ACK, ACK sequence, the communications are immediately sent over to the recipient. Like TCP, we can follow UDP traffic in Wireshark, and inspect its contents.

<img width="597" height="632" alt="image" src="https://github.com/user-attachments/assets/33a7c0ba-c42c-4a7a-a86d-c1cd322af28a" />

### Common Uses of UDP

UDP although less reliable than TCP provides quicker connections through its connectionless state. As such, we might find legitimate traffic that uses UDP like the following:

| Step | Description |
|---|---|
| 1. Real-time Applications | Applications like streaming media, online gaming, real-time voice and video communications |
| 2. DNS (Domain Name System) | DNS queries and responses use UDP |
| 3. DHCP (Dynamic Host Configuration Protocol) | DHCP uses UDP to assign IP addresses and configuration information to network devices. |
| 4. SNMP (Simple Network Management Protocol) | SNMP uses UDP for network monitoring and management |
| 5. TFTP (Trivial File Transfer Protocol) | TFTP uses UDP for simple file transfers, commonly used by older Windows systems and others. |

## 🔑 Key Takeaway

Telnet and UDP traffic can be used for legitimate communication, but attackers may abuse them for data exfiltration and tunneling. Monitor traditional Telnet traffic on port 23, unusual TCP ports carrying Telnet, Telnet over IPv6, and unexpected UDP communications. Inspect suspicious traffic by examining packet contents and following the relevant TCP or UDP stream.
