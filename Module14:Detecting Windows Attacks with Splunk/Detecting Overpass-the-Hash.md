# Detecting Overpass-the-Hash

## Overpass-the-Hash

Adversaries may utilize the Overpass-the-Hash technique to obtain Kerberos TGTs by leveraging stolen password hashes to move laterally within an environment or to bypass typical system access controls. Overpass-the-Hash (also known as Pass-the-Key) allows authentication to occur via Kerberos rather than NTLM. Both NTLM hashes or AES keys can serve as a basis for requesting a Kerberos TGT.

### Attack Steps:

1. The attacker employs tools such as Mimikatz to extract the NTLM hash of a user who is currently logged in to the compromised system. The attacker must have at least local administrator privileges on the system to be able to extract the hash of the user.

<img width="962" height="476" alt="image" src="https://github.com/user-attachments/assets/28ba57ce-d99b-4cb5-a5ea-2a49493b727e" />

2. The attacker uses a tool such as Rubeus to craft a raw AS-REQ request for a specified user to request a TGT ticket. This step does not require elevated privileges on the host to request the TGT, which makes it a stealthier approach than the Mimikatz Pass-the-Hash attack.

<img width="882" height="376" alt="image" src="https://github.com/user-attachments/assets/c9e31983-c272-41ac-9c62-3939fc2d4655" />

3. Analogous to the Pass-the-Ticket technique, the attacker submits the requested ticket for the current logon session.

## Overpass-the-Hash Detection Opportunities

Mimikatz's Overpass-the-Hash attack leaves the same artifacts as the Pass-the-Hash attack, and can be detected using the same strategies.

Rubeus, however, presents a somewhat different scenario. Unless the requested TGT is used on another host, Pass-the-Ticket detection mechanisms may not be effective, as Rubeus sends an AS-REQ request directly to the Domain Controller (DC), generating Event ID 4768 (Kerberos TGT Request). However, communication with the DC (TCP/UDP port 88) from an unusual process can serve as an indicator of a potential Overpass-the-Hash attack.

## Detecting Overpass-the-Hash With Splunk (Targeting Rubeus)

Now let's explore how we can identify Overpass-the-Hash, using Splunk.

**Timeframe:** `earliest=1690443407 latest=1690443544`

```shellsession id="p3x8kw"
index=main earliest=1690443407 latest=1690443544 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" (EventCode=3 dest_port=88 Image!=*lsass.exe) OR EventCode=1
| eventstats values(process) as process by process_id
| where EventCode=3
| stats count by _time, Computer, dest_ip, dest_port, Image, process
| fields - count
```

<img width="1673" height="610" alt="image" src="https://github.com/user-attachments/assets/e93bb6dd-fcb3-4508-9a5e-860be5882076" />

## 🔑 Key Takeaway

Overpass-the-Hash allows attackers to obtain Kerberos TGTs using stolen NTLM hashes or AES keys instead of authenticating through NTLM. Mimikatz-based attacks leave artifacts similar to Pass-the-Hash, while Rubeus can generate Event ID 4768 by sending an AS-REQ directly to the Domain Controller. Monitoring unusual processes communicating with the DC over TCP/UDP port 88 can help identify potential Rubeus-based attacks.
