# TCP Handshake Abnormalities

Innately, when attackers are gaining information on our TCP services, we might notice a few odd behaviors during our traffic analysis efforts. Firstly, let's consider how normal TCP connections work with their 3-way handshake.

<img width="368" height="178" alt="image" src="https://github.com/user-attachments/assets/608f5e3a-c5da-4dee-b73a-1ac062b65ad8" />

To initiate a TCP connection for whatever purpose the client first sends the machine it is attempting to connect to a TCP SYN request to begin the TCP connection.

If this port is open, and in fact able to be connected to, the machine responds with a TCP SYN/ACK to acknowledge that the connection is valid and able to be used. However, we should consider all TCP flags.

## TCP Flags

| Flags | Description |
|---|---|
| URG (Urgent) | This flag is to denote urgency with the current data in stream. |
| ACK (Acknowledgement) | This flag acknowledges receipt of data. |
| PSH (Push) | This flag instructs the TCP stack to immediately deliver the received data to the application layer, and bypass buffering. |
| RST (Reset) | This flag is used for termination of the TCP connection (we will dive into hijacking and RST attacks soon). |
| SYN (Synchronize) | This flag is used to establish an initial connection with TCP. |
| FIN (Finish) | This flag is used to denote the finish of a TCP connection. It is used when no more data needs to be sent. |
| ECN (Explicit Congestion Notification) | This flag is used to denote congestion within our network, it is to let the hosts know to avoid unnecessary re-transmissions. |

As such, when we are performing our traffic analysis efforts we can look for the following strange conditions:

- **Too many flags of a kind or kinds** - This could show us that scanning is occurring within our network.
- **The usage of different and unusual flags** - Sometimes this could indicate a TCP RST attack, hijacking, or simply some form of control evasion for scanning.
- **Solo host to multiple ports, or solo host to multiple hosts** - Easy enough, we can find scanning as we have done before by noticing where these connections are going from one host. In a lot of cases, we may even need to consider decoy scans and random source attacks.

## Excessive SYN Flags

**Related PCAP File(s):**

- `nmap_syn_scan.pcapng`

Right away one of the traffic patterns that we can notice is too many SYN flags. This is a prime example of nmap scanning. Simply put, the adversary will send TCP SYN packets to the target ports. In the case where our port is open, our machine will respond with a SYN-ACK packet to continue the handshake, which will then be met by an RST from the attackers scanner. However, we can get lost in the RSTs here as our machine will respond with RST for closed ports.

<img width="976" height="362" alt="image" src="https://github.com/user-attachments/assets/5c47bbbf-62fb-4a79-b7be-9e288ad229d7" />

However it is worth noting that there are two primary scan types we might detect that use the SYN flag. These are:

- **SYN Scans** - In these scans the behavior will be as we see, however the attacker will pre-emptively end the handshake with the RST flag.
- **SYN Stealth Scans** - In this case the attacker will attempt to evade detection by only partially completing the TCP handshake.

## No Flags

**Related PCAP File(s):**

- `nmap_null_scan.pcapng`

On the opposite side of things, the attacker might send no flags. This is what is commonly referrred to as a NULL scan. In a NULL scan an attacker sends TCP packets with no flags. TCP connections behave like the following when a NULL packet is received.

- **If the port is open** - The system will not respond at all since there is no flags.
- **If the port is closed** - The system will respond with an RST packet.

As such a NULL scan might look like the following.

<img width="935" height="410" alt="image" src="https://github.com/user-attachments/assets/75ab6041-1486-48bd-a77f-7b62261b41c0" />

## Too Many ACKs

**Related PCAP File(s):**

- `nmap_ack_scan.pcapng`

On the other hand, we might notice an excessive amount of acknowledgements between two hosts. In this case the attacker might be employing the usage of an ACK scan. In the case of an ACK scan TCP connections will behave like the following.

- **If the port is open** - The affected machine will either not respond, or will respond with an RST packet.
- **If the port is closed** - The affected machine will respond with an RST packet.

So, we might see the following traffic which would indicate an ACK scan.

<img width="948" height="375" alt="image" src="https://github.com/user-attachments/assets/798080b2-deea-470d-9dcd-b185e1f5892c" />

## Excessive FINs

**Related PCAP File(s):**

- `nmap_fin_scan.pcapng`

Using another part of the handshake, an attacker might utilize a FIN scan. In this case, all TCP packets will be marked with the FIN flag. We might notice the following behavior from our affected machine.

- **If the port is open** - Our affected machine simply will not respond.
- **If the port is closed** - Our affected machine will respond with an RST packet.

<img width="960" height="393" alt="image" src="https://github.com/user-attachments/assets/035b7bdf-c368-44a7-b38a-605f82945cf6" />

## Just too many flags

**Related PCAP File(s):**

- `nmap_xmas_scan.pcapng`

Let's say the attacker just wanted to throw spaghetti at the wall. In that case, they might utilize a Xmas tree scan, which is when they put all TCP flags on their transmissions. Similarly, our affected host might respond like the following when all flags are set.

- **If the port is open** - The affected machine will not respond, or at least it will with an RST packet.
- **If the port is closed** - The affected machine will respond with an RST packet.

Xmas tree scans are pretty easy to spot and look like the following.

<img width="1019" height="376" alt="image" src="https://github.com/user-attachments/assets/324faa8f-d00b-425c-9696-7bfbd486c24c" />

## 🔑 Key Takeaway

TCP handshake abnormalities can reveal port-scanning activity and attempts to evade detection. Monitoring unusual TCP flag patterns—including excessive SYN, ACK, FIN, NULL, and Xmas tree scan traffic—can help identify suspicious connections and distinguish scanning behavior based on how hosts respond to packets.
