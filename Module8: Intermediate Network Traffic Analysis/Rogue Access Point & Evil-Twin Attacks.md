# Rogue Access Point & Evil-Twin Attacks

**Related PCAP File(s):**

- `rogueap.cap`

Addressing rogue access points and evil-twin attacks can seem like a gargantuan task due to their often elusive nature. Nevertheless, with the appropriate strategies in place, these illegitimate access points can be detected and managed effectively. In the realm of malevolent access points, rogue and evil-twin attacks invariably surface as significant concerns.

<img width="588" height="354" alt="image" src="https://github.com/user-attachments/assets/e8062e0d-e0b5-41dd-b641-08c5cf49787d" />

A rogue access point primarily serves as a tool to circumvent perimeter controls in place. An adversary might install such an access point to sidestep network controls and segmentation barriers, which could, in many cases, take the form of hotspots or tethered connections. These rogue points have even been known to infiltrate air-gapped networks. Their primary function is to provide unauthorized access to restricted sections of a network. The critical point to remember here is that rogue access points are directly connected to the network.

## Evil-Twin

An evil-twin on the other hand is spun up by an attacker for many other different purposes. The key here, is that in most cases these access points are not connected to our network. Instead, they are standalone access points, which might have a web server or something else to act as a man-in-the-middle for wireless clients.

<img width="710" height="464" alt="image" src="https://github.com/user-attachments/assets/5440e033-5f9f-4374-831b-bce6b7a34110" />

Attackers might set these up to harvest wireless or domain passwords among other pieces of information. Commonly, these attacks might also encompass a hostile portal attack.

## Airodump-ng Detection

Right away, we could utilize the ESSID filter for Airodump-ng to detect Evil-Twin style access points.

```shell
@htb[/htb]$ sudo airodump-ng -c 4 --essid HTB-Wireless wlan0 -w raw
```

<img width="809" height="199" alt="image" src="https://github.com/user-attachments/assets/0a6ecdbc-ad40-4a43-9f35-12e40db3203f" />

The above example would show that in fact an attacker might have spun up an open access point that has an identical ESSID as our access point. An attacker might do this to host what is commonly referred to as a hostile portal attack. A hostile portal attack is used by attackers in order extract credentials from users among other nefarious actions.

We might also want to be vigilant about deauthentication attempts, which could suggest enforcement measures from the attacker operating the evil-twin access point.

To conclusively ascertain whether this is an anomaly or an Airodump-ng error, we can commence our traffic analysis efforts (rogueap.cap). To filter for beacon frames, we could use the following.

```text
(wlan.fc.type == 00) and (wlan.fc.type_subtype == 8)
```

<img width="1138" height="125" alt="image" src="https://github.com/user-attachments/assets/f9e41c9c-9669-4a9b-b883-cdb1edd06e1b" />

Beacon analysis is crucial in differentiating between genuine and fraudulent access points. One of the initial places to start is the Robust Security Network (RSN) information. This data communicates valuable information to clients about the supported ciphers, among other things.

Suppose we wish to examine our legitimate access point's RSN information.

<img width="731" height="425" alt="image" src="https://github.com/user-attachments/assets/1a0bcf6f-5874-4d60-8280-1491f4ef6069" />

It would indicate that WPA2 is supported with AES and TKIP with PSK as its authentication mechanism. However, when we switch to the illegitimate access point's RSN information, we may find it conspicuously missing.

<img width="622" height="211" alt="image" src="https://github.com/user-attachments/assets/4d3c201b-6321-4fa5-88f6-123c0db0209b" />

In most instances, a standard evil-twin attack will exhibit this characteristic. Nevertheless, we should always probe additional fields for discrepancies, particularly when dealing with more sophisticated evil-twin attacks. For example, an attacker might employ the same cipher that our access point uses, making the detection of this attack more challenging.

Under such circumstances, we could explore other aspects of the beacon frame, such as vendor-specific information, which is likely absent from the attacker's access point.

## Finding a Fallen User

Despite comprehensive security awareness training, some users may fall prey to attacks like these. Fortunately, in the case of open network style evil-twin attacks, we can view most higher-level traffic in an unencrypted format. To filter exclusively for the evil-twin access point, we would employ the following filter.

```text
(wlan.bssid == F8:14:FE:4D:E6:F2)
```

<img width="1157" height="224" alt="image" src="https://github.com/user-attachments/assets/3996ca4a-a9b7-42be-93b1-51d64f99ac63" />

If we detect ARP requests emanating from a client device connected to the suspicious network, we would identify this as a potential compromise indicator. In such instances, we should record pertinent details about the client device to further our incident response efforts.

- Its MAC address
- Its host name

Consequently, we might be able to instigate password resets and other reactive measures to prevent further infringement of our environment.

## Finding Rogue Access Points

On the other hand, detecting rogue access points can often be a simple task of checking our network device lists. In the case of hotspot-based rogue access points (such as Windows hotspots), we might scrutinize wireless networks in our immediate vicinity. If we encounter an unrecognizable wireless network with a strong signal, particularly if it lacks encryption, this could indicate that a user has established a rogue access point to navigate around our perimeter controls.

## 🔑 Key Takeaway

Rogue access points and evil-twin attacks both threaten wireless network security, but they differ in how they connect to the target network. Rogue access points are directly connected to the network and can bypass perimeter controls, while evil-twin access points typically operate independently to deceive wireless clients and harvest credentials. Airodump-ng and Wireshark can help identify suspicious access points by examining ESSIDs, beacon frames, RSN information, encryption settings, and client traffic. Recording affected client details and initiating appropriate response measures can help contain potential compromises.
