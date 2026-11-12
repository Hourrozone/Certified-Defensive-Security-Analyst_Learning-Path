<img width="1049" height="623" alt="image" src="https://github.com/user-attachments/assets/c8c45dfd-234d-447c-840b-8769186a4fcc" /># Detecting Pass-the-Ticket

## Pass-the-Ticket

Pass-the-Ticket (PtT) is a lateral movement technique used by attackers to move laterally within a network by abusing Kerberos TGT (Ticket Granting Ticket) and TGS (Ticket Granting Service) tickets. Instead of using NTLM hashes, PtT leverages Kerberos tickets to authenticate to other systems and access network resources without needing to know the users' passwords. This technique allows attackers to move laterally and gain unauthorized access across multiple systems.

### Attack Steps:

1. The attacker gains administrative access to a system, either through an initial compromise or privilege escalation.

2. The attacker uses tools such as Mimikatz or Rubeus to extract valid TGT or TGS tickets from the compromised system's memory.

<img width="1049" height="623" alt="image" src="https://github.com/user-attachments/assets/15365916-2fa1-4f6c-b5ba-9210dc06368b" />

3. The attacker submits the extracted ticket for the current logon session. The attacker can now authenticate to other systems and network resources without needing plaintext passwords.

<img width="829" height="83" alt="image" src="https://github.com/user-attachments/assets/f98eb6a3-7e76-42a1-b2f2-78d3eabcfd1f" />

<img width="785" height="270" alt="image" src="https://github.com/user-attachments/assets/1d535c99-30bc-4b76-8123-5711815bcc94" />

## Kerberos Authentication Process

Kerberos is a network authentication protocol used to securely authenticate users and services within a Windows Active Directory (AD) environment. The following steps occur in the Kerberos authentication process:

1. The user (client) initiates the authentication process by requesting a Ticket Granting Ticket (TGT) from the Key Distribution Center (KDC), typically part of the Active Directory domain controller.

2. The KDC verifies the user's identity (usually through a password) and issues a TGT encrypted with the user's secret key. The TGT is valid for a specific period and allows the user to request service tickets without needing to re-authenticate.

3. The client sends a service ticket request (TGS-REQ) to the KDC for the service using the TGT obtained in the previous step.

4. The KDC validates the client's TGT and, if successful, issues a service ticket (TGS) encrypted with the service account's secret key and containing the client's identity and a session key. The client then receives the service ticket (TGS) from the KDC.

5. The client connects to the server and sends the TGS to the server as part of the authentication process.

<img width="490" height="292" alt="image" src="https://github.com/user-attachments/assets/b30a00c2-0abe-4d78-b751-bae84b388d52" />

## Related Windows Security Events

During user access to network resources, several Windows Event Logs are generated to record the logon process and related activities.

- **Event ID 4648 (Explicit Credential Logon Attempt):** This event is logged when explicit credentials (e.g., username and password) are provided during logon.

- **Event ID 4624 (Logon):** This event indicates that a user has successfully logged on to the system.

- **Event ID 4672 (Special Logon):** This event is logged when a user's logon includes special privileges, such as running applications as an administrator.

- **Event ID 4768 (Kerberos TGT Request):** This event is logged when a client requests a Ticket Granting Ticket (TGT) during the Kerberos authentication process.

- **Event ID 4769 (Kerberos Service Ticket Request):** When a client requests a Service Ticket (TGS Ticket) to access a remote service during the Kerberos authentication process, Event ID 4769 is generated.

<img width="1285" height="578" alt="image" src="https://github.com/user-attachments/assets/45b9426a-e55a-40da-9eca-32dd94ad672c" />

## Pass-the-Ticket Detection Opportunities

Detecting Pass-the-Ticket attacks can be challenging, as attackers are leveraging valid Kerberos tickets instead of traditional credential hashes. The key distinction is that when the Pass-the-Ticket attack is executed, the Kerberos Authentication process will be partial. For example, an attacker imports a TGT ticket into a logon session and requests a TGS ticket for a remote service. From the Domain Controller perspective, the imported TGT was never requested before from the attacker’s system, so there won't be an associated Event ID 4768.

<img width="660" height="430" alt="image" src="https://github.com/user-attachments/assets/173d1c6c-3ff3-4736-b193-61d3aa940753" />

This approach can be converted into the following Splunk detection: Look for Event ID 4769 (Kerberos Service Ticket Request) or Event ID 4770 (Kerberos Service Ticket was renewed) without a prior Event ID 4768 (Kerberos TGT Request) from the same system within a specific time window.

Another approach is looking for mismatches between Service and Host IDs (in Event ID 4769) and the actual Source and Destination IPs (in Event ID 3). Note that there will be several legitimate mismatches, but unusual hostnames or services should be investigated further.

Also, in cases where an attacker imports a TGS ticket into the logon session, it is important to review Event ID 4771 (Kerberos Pre-Authentication Failed) for mismatches between Pre-Authentication type and Failure Code. For example, Pre-Authentication type 2 (Encrypted Timestamp) with Failure Code 0x18 (Pre-authentication information was invalid) would indicate that the client sent a Kerberos AS-REQ with a pre-authentication encrypted timestamp, but the KDC couldn’t decrypt it.

It is essential to understand that these detection opportunities should be enhanced with behavior-based detection. In other words, context is vital. Looking for Event IDs 4769, 4770, or 4771 alone will likely generate many false positives. Correlate the event logs with user and system behavior patterns, and consider whether there are any suspicious activities associated with the user or system involved in the logs.

<img width="1667" height="688" alt="image" src="https://github.com/user-attachments/assets/69c4823f-f84b-4020-a29a-2bd5133fb8ea" />

## Detecting Pass-the-Ticket With Splunk

Now let's explore how we can identify Pass-the-Ticket, using Splunk.

**Timeframe:** `earliest=1690451665 latest=1690451745`

```shellsession
index=main earliest=1690392405 latest=1690451745 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

(image here)

### Search Breakdown:

- **`index=main earliest=1690392405 latest=1690451745 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770)`:** This command filters events from the main index that fall within the specified time range. It selects events from the WinEventLog:Security source, where the user field does not end with a dollar ($) and the EventCode is one of 4768, 4769, or 4770.

- **`| rex field=user "(?<username>[^@]+)"`:** This command extracts the username from the user field using a regular expression. It assigns the extracted value to a new field called username.

- **`| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"`:** This command extracts the IPv4 address from the src_ip field, even if it's originally recorded as an IPv6 address. It assigns the extracted value to a new field called src_ip_4.

- **`| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)`:** This command groups events into transactions based on the username and src_ip_4 fields. A transaction begins with an event that has an EventCode of 4768. The maxspan=10h parameter sets a maximum duration of 10 hours for a transaction. The keepevicted=true parameter ensures that open transactions without an ending event are included in the results.

- **`| where closed_txn=0`:** This command filters the results to include only open transactions, which do not have an ending event.

- **`| search NOT user="*$@*"`:** This command filters out results where the user field ends with an asterisk (*) and contains an at sign (@).

- **`| table _time, ComputerName, username, src_ip_4, service_name, category`:** This command displays the specified fields in a table format.

## 🔑 Key Takeaway

Pass-the-Ticket abuses valid Kerberos TGT or TGS tickets to authenticate to remote systems without knowing the user's password. A key detection opportunity is identifying Kerberos service ticket requests (Event ID 4769) or renewals (Event ID 4770) without a preceding TGT request (Event ID 4768) from the same system within a defined time window. Correlating these events with system and user behavior helps reduce false positives and identify suspicious lateral movement.
