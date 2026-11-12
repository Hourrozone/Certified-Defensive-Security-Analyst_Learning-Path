# Detecting Responder-like Attacks

## LLMNR/NBT-NS/mDNS Poisoning

LLMNR (Link-Local Multicast Name Resolution) and NBT-NS (NetBIOS Name Service) poisoning, also referred to as NBNS spoofing, are network-level attacks that exploit inefficiencies in these name resolution protocols. Both LLMNR and NBT-NS are used to resolve hostnames to IP addresses on local networks when the fully qualified domain name (FQDN) resolution fails. However, their lack of built-in security mechanisms renders them susceptible to spoofing and poisoning attacks.

Typically, attackers employ the Responder tool to execute LLMNR, NBT-NS, or mDNS poisoning.

### Attack Steps

1. A victim device sends a name resolution query for a mistyped hostname (e.g., `fileshrae`).
2. DNS fails to resolve the mistyped hostname.
3. The victim device sends a name resolution query for the mistyped hostname using LLMNR/NBT-NS.
4. The attacker's host responds to the LLMNR (UDP 5355)/NBT-NS (UDP 137) traffic, pretending to know the identity of the requested host. This effectively poisons the service, directing the victim to communicate with the adversary-controlled system.

<img width="542" height="301" alt="image" src="https://github.com/user-attachments/assets/7dc245b0-8a9e-4213-8f97-faeeb8da6a11" />

The result of a successful attack is the acquisition of the victim's NetNTLM hash, which can be either cracked or relayed in an attempt to gain access to systems where these credentials are valid.

## Responder Detection Opportunities

Detecting LLMNR, NBT-NS, and mDNS poisoning can be challenging. However, organizations can mitigate the risk by implementing the following measures:

- Deploy network monitoring solutions to detect unusual LLMNR and NBT-NS traffic patterns, such as an elevated volume of name resolution requests from a single source.
- Employ a honeypot approach - name resolution for non-existent hosts should fail. If an attacker is present and spoofing LLMNR/NBT-NS/mDNS responses, name resolution will succeed. https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/

<img width="1064" height="552" alt="image" src="https://github.com/user-attachments/assets/4ddb3b38-fe73-450e-bf15-5214c2479e67" />

A PowerShell script similar to the above can be automated to run as a scheduled task to aid in detection. Logging this activity might pose a challenge, but the `New-EventLog` PowerShell cmdlet can be used.

```powershell
PS C:\Users\Administrator> New-EventLog -LogName Application -Source LLMNRDetection
```

To create an event, the `Write-EventLog` cmdlet should be used:

```powershell
PS C:\Users\Administrator> Write-EventLog -LogName Application -Source LLMNRDetection -EventId 19001 -Message $msg -EntryType Warning
```

## Detecting Responder-like Attacks With Splunk

Now let's explore how we can identify the Responder-like attacks previously discussed, using Splunk and logs from a PowerShell script similar to the one above.

**Timeframe:** `earliest=1690290078 latest=1690291207`

```shell
index=main earliest=1690290078 latest=1690291207 SourceName=LLMNRDetection
| table _time, ComputerName, SourceName, Message
```

<img width="1887" height="441" alt="image" src="https://github.com/user-attachments/assets/342db2c6-251e-4861-b79a-3432f804b4cc" />

Sysmon Event ID 22 can also be utilized to track DNS queries associated with non-existent/mistyped file shares.

**Timeframe:** `earliest=1690290078 latest=1690291207`

```shell
index=main earliest=1690290078 latest=1690291207 EventCode=22 
| table _time, Computer, user, Image, QueryName, QueryResults
```

<img width="1348" height="189" alt="image" src="https://github.com/user-attachments/assets/36ef66e9-c5f8-48ef-912a-3192518c6e39" />

Additionally, remember that Event 4648 can be used to detect explicit logons to rogue file shares which attackers might use to gather legitimate user credentials.

**Timeframe:** `earliest=1690290814 latest=1690291207`

```shell
index=main earliest=1690290814 latest=1690291207 EventCode IN (4648) 
| table _time, EventCode, source, name, user, Target_Server_Name, Message
| sort 0 _time
```

<img width="1882" height="1374" alt="image" src="https://github.com/user-attachments/assets/7faa9ccd-8cb7-40d7-9a2f-bc281821fe6c" />

---

## 🔑 Key Takeaway

Responder-like attacks exploit insecure name resolution protocols such as LLMNR, NBT-NS, and mDNS to redirect victims to attacker-controlled systems and capture NetNTLM hashes. Detection can involve monitoring unusual name resolution traffic, using honeypots for non-existent hostnames, and collecting PowerShell-generated events. In Splunk, analysts can correlate custom detection events, Sysmon Event ID 22 DNS queries, and Security Event ID 4648 to identify suspicious name resolution and explicit credential usage.
