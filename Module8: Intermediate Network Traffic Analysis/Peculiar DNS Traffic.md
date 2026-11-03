# Peculiar DNS Traffic

DNS Traffic can be cumbersome to inspect, as many times our clients will generate a ton of it, and abnormalities can sometimes get buried in the mass volume of it. However, understanding DNS and some direct signs of malicious actions is important in our traffic analysis efforts.

## DNS Queries

DNS queries are used when a client wants to resolve a domain name with an IP address, or the other way around. First, we can explore the most common type of query, which is forward lookups.

<img width="568" height="355" alt="image" src="https://github.com/user-attachments/assets/e696b269-01e9-40e1-b113-bbb2d71062cb" />

Generally speaking, when a client initiates a DNS forward lookup query, it does the following steps.

**Request:**

Where is `academy.hackthebox.com`?

**Response:**

Well its at `192.168.10.6`

| Step | Description |
|---|---|
| 1. Query Initiation | When the user wants to visit something like academy.hackthebox.com it initiates a DNS forward query. |
| 2. Local Cache Check | The client then checks its local DNS cache to see if it has already resolved the domain name to an IP address. If not it continues with the following. |
| 3. Recursive Query | The client then sends its recursive query to its configured DNS server (local or remote). |
| 4. Root Servers | The DNS resolver, if necessary, starts by querying the root name servers to find the authoritative name servers for the top-level domain (TLD). There are 13 root servers distributed worldwide. |
| 5. TLD Servers | The root server then responds with the authoritative name servers for the TLD (aka .com or .org) |
| 6. Authoritative Servers | The DNS resolver then queries the TLD's authoritative name servers for the second-level domain (aka hackthebox.com). |
| 7. Domain Name's Authoritative Servers | Finally, the DNS resolver queries the domains authoritative name servers to obtain the IP address associated with the requested domain name (aka academy.hackthebox.com). |
| 8. Response | The DNS resolver then receives the IP address (A or AAAA record) and sends it back to the client that initiated the query. |

## DNS Reverse Lookups/Queries

On the opposite side, we have Reverse Lookups. These occur when a client already knows the IP address and wants to find the corresponding FQDN (Fully Qualified Domain Name).

**Request:**

What is your name `192.168.10.6`?

**Response:**

Well its academy.hackthebox.com :)

In this case the steps are a bit less complicated.

| Step | Description |
|---|---|
| 1. Query Initiation | The client sends a DNS reverse query to its configured DNS resolver (server) with the IP address it wants to find the domain name. |
| 2. Reverse Lookup Zones | The DNS resolver checks if it is authoritative for the reverse lookup zone that corresponds to the IP range as determined by the received IP address. Aka 192.0.2.1, the reverse zone would be 1.2.0.192.in-addr.arpa |
| 3. PTR Record Query | The DNS resolver then looks for a PTR record on the reverse lookup zone that corresponds to the provided IP address. |
| 4. Response | If a matching PTR is found, the DNS server (resolver) then returns the FQDN of the IP for the client. |

<img width="837" height="410" alt="image" src="https://github.com/user-attachments/assets/300708da-b412-46af-aef8-f9114536222f" />

## DNS Record Types

DNS has many different record types responsible for holding different information. We should be familiar with these, especially when monitoring DNS traffic.

| Record Type | Description |
|---|---|
| A (Address) | This record maps a domain name to an IPv4 address |
| AAAA (Ipv6 Address) | This record maps a domain name to an IPv6 address |
| CNAME (Canonical Name) | This record creates an alias for the domain name. Aka hello.com = world.com |
| MX (Mail Exchange) | This record specifies the mail server responsible for receiving email messages on behalf of the domain. |
| NS (Name Server) | This specifies an authoritative name servers for a domain. |
| PTR (Pointer) | This is used in reverse queries to map an IP to a domain name |
| TXT (Text) | This is used to specify text associated with the domain |
| SOA (Start of Authority) | This contains administrative information about the zone |

## Finding DNS Enumeration Attempts

**Related PCAP File(s):**

- `dns_enum_detection.pcapng`

We might notice a significant amount of DNS traffic from one host when we start to look at our raw output in Wireshark.

```text
dns
```

<img width="803" height="391" alt="image" src="https://github.com/user-attachments/assets/ab5d0b94-6fe6-46c9-b815-8deef002bca3" />

We might even notice this traffic concluded with something like ANY:

<img width="971" height="71" alt="image" src="https://github.com/user-attachments/assets/7e3e5e53-74b1-409a-a700-72e80140cfe0" />

This would be a clear indication of DNS enumeration and possibly even subdomain enumeration from an attacker.

## Finding DNS Tunneling

**Related PCAP File(s):**

- `dns_tunneling.pcapng`

On the other hand, we might notice a good amount of text records from one host. This could indicate DNS tunneling. Like ICMP tunneling, attackers can and have utilized DNS forward and reverse lookup queries to perform data exfiltration. They do so by appending the data they would like to exfiltrate as a part of the TXT field.

If this was happening it might look like the following.

<img width="1016" height="325" alt="image" src="https://github.com/user-attachments/assets/747533cf-4749-4222-b2eb-740980b0f42e" />

If we were to dig a little deeper, we might notice some out of place text on the lower right-hand side of our screen.

<img width="552" height="140" alt="image" src="https://github.com/user-attachments/assets/ba5e9ac3-000f-4645-9c49-87b2817e9f8e" />

However, in many cases, this data might be encoded or encrypted, and we might notice the following.

<img width="540" height="208" alt="image" src="https://github.com/user-attachments/assets/3fefaca7-eca9-481c-9379-8e2084c56a4d" />

We can retrieve this value from wireshark by locating it like the following and right-clicking the value to specify to copy it.



Then if we were to go into our Linux machine, in this case we could utilize something like base64 -d to retrieve the true value.

```shell
@htb[/htb]$ echo 'VTBaU1EyVXhaSFprVjNocldETnNkbVJXT1cxaU0wb3pXVmhLYTFneU1XeFlNMUp2WVZoT1ptTklTbXhrU0ZJMVdETkNjMXBYUm5wYQpXREJMQ2c9PQo=' | base64 -d 

U0ZSQ2UxZHZkV3hrWDNsdmRWOW1iM0ozWVhKa1gyMWxYM1JvYVhOZmNISmxkSFI1WDNCc1pXRnpaWDBLCg==
```

However, in some cases attackers will double if not triple encode the value they are attempting to exfiltrate through DNS tunneling, so we might need to do the following.

```shell
@htb[/htb]$ echo 'VTBaU1EyVXhaSFprVjNocldETnNkbVJXT1cxaU0wb3pXVmhLYTFneU1XeFlNMUp2WVZoT1ptTklTbXhrU0ZJMVdETkNjMXBYUm5wYQpXREJMQ2c9PQo=' | base64 -d | base64 -d | base64 -d
```

However, we might need to do more than just base64 decode these values, as in many cases as mentioned these values might be encrypted.

Attackers might conduct DNS tunneling for the following reasons:

| Step | Description |
|---|---|
| 1. Data Exfiltration | As shown above DNS tunneling can be helpful for attackers trying to get data out of our network without getting caught. |
| 2. Command and Control | Some malware and malicious agents will utilize DNS tunneling on compromised systems in order to communicate back to their command and control servers. Notably, we might see this method of usage in botnets. |
| 3. Bypassing Firewalls and Proxies | DNS tunneling allows attackers to bypass firewalls and web proxies that only monitor HTTP/HTTPs traffic. DNS traffic is traditionally allowed to pass through network boundaries. As such, it is important that we monitor and control this traffic. |
| 4. Domain Generation Algorithms (DGAs) | Some more advanced malware will utilize DNS tunnels to communicate back to their command and control servers that use dynamically generated domain names through DGAs. This makes it much more difficult for us to detect and block these domain names. |

## The Interplanetary File System and DNS Tunneling

It has been observed in recent years that advanced threat actors will utilize the Interplanetary file System to store and pull malicious files. As such we should always watch out for DNS and HTTP/HTTPs traffic to URIs like the following:

```text
https://cloudflare-ipfs.com/ipfs/QmS6eyoGjENZTMxM7UdqBk6Z3U3TZPAVeJXdgp9VK4o1Sz
```

These forms of attacks can be exceptionally difficult to detect as IPFS innately operates on a peer to peer basis. To learn more, we can research into IPFS.

**Interplanetary File System**

## 🔑 Key Takeaway

DNS traffic analysis can reveal enumeration, subdomain discovery, and tunneling attempts. Unusual query volumes, ANY queries, suspicious TXT records, unusually long or encoded DNS data, and unexpected IPFS-related traffic can indicate malicious activity. Monitoring DNS requests and inspecting suspicious payloads can help detect data exfiltration and command-and-control communications.
