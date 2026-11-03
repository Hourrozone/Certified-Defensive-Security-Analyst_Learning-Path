# TCP Connection Resets & Hijacking

Unfortunately, TCP does not provide the level of protection to prevent our hosts from having their connections terminated or hijacked by an attacker. As such, we might notice that a connection gets terminated by an RST packet, or hijacked through connection hijacking.

## TCP Connection Termination

**Related PCAP File(s):**

- `RST_Attack.pcapng`

Suppose an adversary wanted to cause denial-of-service conditions within our network. They might employ a simple TCP RST Packet injection attack, or TCP connection termination in simple terms.

This attack is a combination of a few conditions:

1. The attacker will spoof the source address to be the affected machine's
2. The attacker will modify the TCP packet to contain the RST flag to terminate the connection
3. The attacker will specify the destination port to be the same as one currently in use by one of our machines.

As such, we might notice an excessive amount of packets going to one port.

<img width="875" height="344" alt="image" src="https://github.com/user-attachments/assets/7aca5733-669f-43c3-a4c9-cdc776c0a0fb" />

One way we can verify that this is indeed a TCP RST attack is through the physical address of the transmitter of these TCP RST packets. Suppose, the IP address `192.168.10.4` is registered to `aa:aa:aa:aa:aa:aa` in our network device list, and we notice an entirely different MAC sending these like the following.

<img width="1031" height="89" alt="image" src="https://github.com/user-attachments/assets/69070496-2813-4bc8-b52c-5f3801f0a725" />

This would indicate malicious activity within our network, and we could conclude that this is likely a TCP RST Attack. However, it is worth noting that an attacker might spoof their MAC address in order to further evade detection. In this case, we could notice retransmissions and other issues as we saw in the ARP poisoning section.

## TCP Connection Hijacking

**Related PCAP File(s):**

- `TCP-hijacking.pcap`

For more advanced actors, they might employ TCP connection hijacking. In this case the attacker will actively monitor the target connection they want to hijack.

1. The attacker will then conduct sequence number prediction in order to inject their malicious packets in the correct order. During this injection they will spoof the source address to be the same as our affected machine.
2. The attacker will need to block ACKs from reaching the affected machine in order to continue the hijacking. They do this either through delaying or blocking the ACK packets. As such, this attack is very commonly employed with ARP poisoning, and we might notice the following in our traffic analysis.

<img width="300" height="50" alt="image" src="https://github.com/user-attachments/assets/73078f71-6ef5-4332-92a2-41d2ec8c2627" />

## 🔑 Key Takeaway

TCP connection termination attacks use spoofed RST packets to disrupt active connections, while TCP connection hijacking involves monitoring a connection, predicting sequence numbers, and injecting malicious packets. Monitoring packet sources, MAC addresses, retransmissions, and unusual ACK behavior can help identify these attacks.
