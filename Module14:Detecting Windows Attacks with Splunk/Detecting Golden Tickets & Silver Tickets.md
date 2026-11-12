# Detecting Golden Tickets/Silver Tickets

## Golden Ticket

A Golden Ticket attack is a potent method where an attacker forges a Ticket Granting Ticket (TGT) to gain unauthorized access to a Windows Active Directory domain as a domain administrator. The attacker creates a TGT with arbitrary user credentials and then uses this forged ticket to impersonate a domain administrator, thereby gaining full control over the domain. The Golden Ticket attack is stealthy and persistent, as the forged ticket has a long validity period and remains valid until it expires or is revoked.

### Attack Steps:

1. The attacker extracts the NTLM hash of the KRBTGT account using a DCSync attack (alternatively, they can use NTDS.dit and LSASS process dumps on the Domain Controller).

<img width="988" height="602" alt="image" src="https://github.com/user-attachments/assets/52f08348-992c-4c99-8aeb-dc5abf3f4347" />

2. Armed with the KRBTGT hash, the attacker forges a TGT for an arbitrary user account, assigning it domain administrator privileges.

<img width="898" height="687" alt="image" src="https://github.com/user-attachments/assets/c749d95e-925c-4253-bd9b-51016cb609ca" />

3. The attacker injects the forged TGT in the same manner as a Pass-the-Ticket attack.

## Golden Ticket Detection Opportunities

Detecting Golden Ticket attacks can be challenging, as the TGT can be forged offline by an attacker, leaving virtually no traces of Mimikatz execution. One option is to monitor common methods of extracting the KRBTGT hash:

- DCSync attack
- NTDS.dit file access
- LSASS memory read on the domain controller (Sysmon Event ID 10)

From another standpoint, a Golden Ticket is just another ticket for Pass-the-Ticket detection.

## Detecting Golden Tickets With Splunk (Yet Another Ticket To Be Passed Approach)

Now let's explore how we can identify Golden Tickets, using Splunk.

**Timeframe:** `earliest=1690451977 latest=1690452262`

```shellsession
index=main earliest=1690451977 latest=1690452262 source="WinEventLog:Security" user!=*$ EventCode IN (4768,4769,4770) 
| rex field=user "(?<username>[^@]+)"
| rex field=src_ip "(\:\:ffff\:)?(?<src_ip_4>[0-9\.]+)"
| transaction username, src_ip_4 maxspan=10h keepevicted=true startswith=(EventCode=4768)
| where closed_txn=0
| search NOT user="*$@*"
| table _time, ComputerName, username, src_ip_4, service_name, category
```

<img width="1001" height="385" alt="image" src="https://github.com/user-attachments/assets/155cf3d2-24a5-42ba-ab90-7fbcc6631f94" />

## Silver Ticket

Adversaries who possess the password hash of a target service account (e.g., SharePoint, MSSQL) may forge Kerberos Ticket Granting Service (TGS) tickets, also known as Silver Tickets. Silver tickets can be used to impersonate any user, but they are more limited in scope than Golden Tickets, as they only allow adversaries to access a specific resource (e.g., MSSQL) and the system hosting the resource.

### Attack Steps:

1. The attacker extracts the NTLM hash of the targeted service account (or the computer account for CIFS access) using tools like Mimikatz or other credential dumping techniques.

2. Generate a Silver Ticket: Using the extracted NTLM hash, the attacker employs tools like Mimikatz to create a forged TGS ticket for the specified service.

<img width="1056" height="502" alt="image" src="https://github.com/user-attachments/assets/fbb5499e-74d5-45bc-8048-9b9c51f75eeb" />

3. The attacker injects the forged TGS in the same manner as a Pass-the-Ticket attack.

<img width="1057" height="513" alt="image" src="https://github.com/user-attachments/assets/15adeebe-f50e-40f5-8b6c-7fb21d849567" />

## Silver Ticket Detection Opportunities

Detecting forged service tickets (TGS) can be challenging, as there are no simple indicators of attack. In both Golden Ticket and Silver Ticket attacks, arbitrary users can be used, including non-existent ones. Event ID 4720 (A user account was created) can help identify newly created users. Subsequently, we can compare this user list with logged-in users.

Because there is no validation for user permissions, users can be granted administrative permissions. Event ID 4672 (Special Logon) can be employed to detect anomalously assigned privileges.

## Detecting Silver Tickets With Splunk

Now let's explore how we can identify Silver Tickets, using Splunk.

### Detecting Silver Tickets With Splunk Through User Correlation

Let's first create a list of users (users.csv) leveraging Event ID 4720 (A user account was created) as follows.

```shellsession
index=main latest=1690448444 EventCode=4720
| stats min(_time) as _time, values(EventCode) as EventCode by user
| outputlookup users.csv
```

**Note:** users.csv can be downloaded from the Resources section of this module (upper right corner) and uploaded to Splunk by clicking Settings -> Lookups -> Lookup table files -> New Lookup Table File.

Let's now compare the list above with logged-in users as follows.

**Timeframe:** `latest=1690545656`

```shellsession
index=main latest=1690545656 EventCode=4624
| stats min(_time) as firstTime, values(ComputerName) as ComputerName, values(EventCode) as EventCode by user
| eval last24h = 1690451977
| where firstTime > last24h
```

```shellsession
| eval last24h=relative_time(now(),"-24h@h")
```

```shellsession
| convert ctime(firstTime)
| convert ctime(last24h)
| lookup users.csv user as user OUTPUT EventCode as Events
| where isnull(Events)
```

<img width="1671" height="707" alt="image" src="https://github.com/user-attachments/assets/97f7c6e3-c1f8-44b9-911f-f556908413a9" />

### Search Breakdown:

- **`index=main latest=1690545656 EventCode=4624`:** This command filters events from the main index that occur before a specified timestamp and have an EventCode of 4624, indicating a successful login.

- **`| stats min(_time) as firstTime, values(ComputerName) as ComputerName, values(EventCode) as EventCode by user`:** This command calculates the earliest login time for each user, groups them by the user field, and creates a table with columns firstTime, ComputerName, and EventCode.

- **`| eval last24h = 1690451977`:** This command defines a variable last24h and assigns it a specific timestamp value. This value represents a time threshold for filtering the results.

- **`| where firstTime > last24h`:** This command filters the results to include only logins that occurred after the time threshold defined in last24h.

- **`| eval last24h=relative_time(now(),"-24h@h")`:** This command (commented out) would redefine the last24h variable to be exactly 24 hours before the current time. Note that this line is commented out with backticks, so it will not be executed in this search.

- **`| convert ctime(firstTime)`:** This command converts the firstTime field from epoch time to a human-readable format.

- **`| convert ctime(last24h)`:** This command converts the last24h field from epoch time to a human-readable format.

- **`| lookup users.csv user as user OUTPUT EventCode as Events`:** This command performs a lookup using the users.csv file, matches the user field from the search results with the user field in the CSV file, and outputs the EventCode column from the CSV file as a new field called Events.

- **`| where isnull(Events)`:** This command filters the results to include only those where the Events field is null. This indicates that the user was not found in the users.csv file.

### Detecting Silver Tickets With Splunk By Targeting Special Privileges Assigned To New Logon

**Timeframe:** `latest=1690545656`

```shellsession
index=main latest=1690545656 EventCode=4672
| stats min(_time) as firstTime, values(ComputerName) as ComputerName by Account_Name
| eval last24h = 1690451977 
```

```shellsession
| eval last24h=relative_time(now(),"-24h@h") 
```

```shellsession
| where firstTime > last24h 
| table firstTime, ComputerName, Account_Name 
| convert ctime(firstTime)
```

<img width="1669" height="625" alt="image" src="https://github.com/user-attachments/assets/740a4355-2a57-44ea-a870-dc9bfc3c811d" />

## 🔑 Key Takeaway

Golden Tickets are forged TGTs created using the KRBTGT account hash, potentially granting domain-wide access. Silver Tickets are forged TGS tickets created using a service account or computer account hash, providing access to specific services. Detection strategies include monitoring KRBTGT hash extraction, correlating Kerberos ticket events, identifying newly created users through Event ID 4720, and investigating unusual privileged logons through Event ID 4672.
