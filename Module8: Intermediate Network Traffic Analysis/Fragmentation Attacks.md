# Fragmentation Attacks

**Related PCAP File(s):**

- `nmap_frag_fw_bypass.pcapng`

When we begin to look for network anomalies, we should always consider the IP layer. Simply put, the IP layer functions in its ability to transfer packets from one hop to another. This layer uses source and destination IP addresses for inter-host communications. When we examine this traffic, we can identify the IP addresses as they exist within the IP header of the packet.

However, it is essential to note that this layer has no mechanisms to identify when packets are lost, dropped, or otherwise tampered with. Instead, we need to recognize that these mishaps are handled by the transport or application layers for this data. To dissect these packets, we can explore some of their fields:

- **Length - IP header length:** This field contains the overall length of the IP header.
- **Total Length - IP Datagram/Packet Length:** This field specifies the entire length of the IP packet, including any relevant data.
- **Fragment Offset:** In many cases when a packet is large enough to be divided, the fragmentation offset will be set to provide instructions to reassemble the packet upon delivery to the destination host.
- **Source and Destination IP Addresses:** These fields contain the origination (source) and destination IP addresses for the two communicating hosts.

<img width="337" height="183" alt="image" src="https://github.com/user-attachments/assets/8affb5dd-5aa4-491c-b18c-8161c11126e6" />

## Commonly Abused Fields

Innately, attackers might craft these packets to cause communication issues. Traditionally, an attacker might attempt to evade IDS controls through packet malformation or modification. As such, diving into each one of these fields and understanding how we can detect their misuse will equip us with the tools to succeed in our traffic analysis efforts.

## Abuse of Fragmentation

Fragmentation serves as a means for our legitimate hosts to communicate large data sets to one another by splitting the packets and reassembling them upon delivery. This is commonly achieved through setting a maximum transmission unit (MTU). The MTU is used as the standard to divide these large packets into equal sizes to accommodate the entire transmission. It is worth noting that the last packet will likely be smaller. This field gives instructions to the destination host on how it can reassemble these packets in logical order.

Commonly, attackers might abuse this field for the following purposes:

- **IPS/IDS Evasion** - Let's say for instance that our intrusion detection controls do not reassemble fragmented packets. Well, for short, an attacker could split their nmap or other enumeration techniques to be fragmented, and as such it could bypass these controls and be reassembled at the destination.
- **Firewall Evasion** - Through fragmentation, an attacker could likewise evade a firewall's controls through fragmentation. Once again, if the firewall does not reassemble these packets before delivery to the destination host, the attacker's enumeration attempt might succeed.
- **Firewall/IPS/IDS Resource Exhaustion** - Suppose an attacker were to craft their attack to fragment packets to a very small MTU (10, 15, 20, and so on), the network control might not reassemble these packets due to resource constraints, and the attacker might succeed in their enumeration efforts.
- **Denial of Service** - For old hosts, an attacker might utilize fragmentation to send IP packets exceeding 65535 bytes through ping or other commands. In doing so, the destination host will reassemble this malicious packet and experience countless different issues. As such, the resultant condition is successful denial-of-service from the attacker.

If our network mechanism were to perform correctly. It should do the following:

- **Delayed Reassembly** - The IDS/IPS/Firewall should act the same as the destination host, in the sense that it waits for all fragments to arrive to reconstruct the transmission to perform packet inspection.

## Finding Irregularities in Fragment Offsets

In order to better understand the abovementioned mechanics, we can open the related traffic capture file in Wireshark.

```shell
@htb[/htb]$ wireshark nmap_frag_fw_bypass.pcapng
```

For starters, we might notice several ICMP requests going to one host from another, this is indicative of the starting requests from a traditional Nmap scan. This is the beginning of the host discovery process. An attacker might run a command like this.

### Attacker's Enumeration

```shell
@htb[/htb]$ nmap <host ip>
```

In doing so, they will generate the following.

<img width="1134" height="495" alt="image" src="https://github.com/user-attachments/assets/2eac8be5-8b25-41ba-8755-e90b33152b61" />

Secondarily, an attacker might define a maximum transmission unit size like this in order to fragment their port scanning packets.

```shell
@htb[/htb]$ nmap -f 10 <host ip>
```

In doing so they will generate IP packets with a maximum size of 10. Seeing a ton of fragmentation from a host can be an indicator of this attack, and it would look like the following.

<img width="1139" height="190" alt="image" src="https://github.com/user-attachments/assets/68d522dc-1d5a-466e-befe-c65868d72c81" />

However, the more notable indicator of a fragmentation scan, regardless of its evasion use is the single host to many ports issues that it generates. Let's take the following for instance.

<img width="1143" height="360" alt="image" src="https://github.com/user-attachments/assets/32f16c29-07bd-4da8-8ef7-fb89158e1c34" />

In this case, the destination host would respond with RST flags for ports which do not have an active service running on them (aka closed ports). This pattern is a clear indication of a fragmented scan.

If our Wireshark is not reassembling packets for our inspection, we can make a quick change in our preferences for the IPv4 protocol.

<img width="693" height="507" alt="image" src="https://github.com/user-attachments/assets/f005e883-1da8-415a-8ec8-940c24e20496" />

## 🔑 Key Takeaway

IP fragmentation allows large packets to be divided into smaller fragments and reassembled at the destination. Attackers can abuse this mechanism to evade IDS/IPS and firewall inspection, exhaust network security resources, or cause denial-of-service conditions. When analyzing traffic in Wireshark, look for unusual fragmentation patterns, small MTU sizes, irregular fragment offsets, and scans targeting multiple ports from a single host. Proper delayed reassembly by network security mechanisms helps ensure fragmented traffic is inspected consistently with how the destination host processes it.
