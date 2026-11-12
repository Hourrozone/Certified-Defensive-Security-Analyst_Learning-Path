# Detecting DCSync & DCShadow

## DCSync

DCSync is a technique exploited by attackers to extract password hashes from Active Directory Domain Controllers (DCs). This method capitalizes on the Replication Directory Changes permission typically granted to domain controllers, enabling them to read all object attributes, including password hashes. Members of the Administrators, Domain Admins, and Enterprise Admin groups, or computer accounts on the domain controller, have the capability to execute DCSync to extract password data from Active Directory. This data may encompass both current and historical hashes of potentially valuable accounts, such as KRBTGT and Administrators.

### Attack Steps:

1. The attacker secures administrative access to a domain-joined system or escalates privileges to acquire the requisite rights to request replication data.

2. Utilizing tools such as Mimikatz, the attacker requests domain replication data by using the DRSGetNCChanges interface, effectively mimicking a legitimate domain controller.

<img width="1006" height="516" alt="image" src="https://github.com/user-attachments/assets/f3329ed7-9ca0-4739-9972-d88fd09e8ffb" />

3. The attacker may then craft Golden Tickets, Silver Tickets, or opt to employ Pass-the-Hash/Overpass-the-Hash attacks.

## DCSync Detection Opportunities

DS-Replication-Get-Changes operations can be recorded with Event ID 4662. However, an additional Audit Policy Configuration is needed since it is not enabled by default (Computer Configuration/Windows Settings/Security Settings/Advanced Audit Policy Configuration/DS Access).

<img width="619" height="723" alt="image" src="https://github.com/user-attachments/assets/628f9e0a-2970-4843-8f47-f5e6672d2ed5" />

Seek out events containing the property `{1131f6aa-9c07-11d1-f79f-00c04fc2dcd2}`, corresponding to DS-Replication-Get-Changes, as Event 4662 solely consists of GUIDs.

## Detecting DCSync With Splunk

Now let's explore how we can identify DCSync, using Splunk.

**Timeframe:** `earliest=1690544278 latest=1690544280`

```shellsession id="v6t4np"
index=main earliest=1690544278 latest=1690544280 EventCode=4662 Message="*Replicating Directory Changes*"
| rex field=Message "(?P<property>Replicating Directory Changes.*)"
| table _time, user, object_file_name, Object_Server, property
```

<img width="1887" height="557" alt="image" src="https://github.com/user-attachments/assets/ab375387-8e69-42cb-8e23-59d5bfa56b99" />

## DCShadow

DCShadow is an advanced tactic employed by attackers to enact unauthorized alterations to Active Directory objects, encompassing the creation or modification of objects without producing standard security logs. The assault harnesses the Directory Replicator (Replicating Directory Changes) permission, customarily granted to domain controllers for replication tasks. DCShadow is a clandestine technique enabling attackers to manipulate Active Directory data and establish persistence within the network. Registration of a rogue DC necessitates the creation of new server and nTDSDSA objects in the Configuration partition of the AD schema, which demands Administrator privileges (either Domain or local to the DC) or the KRBTGT hash.

### Attack Steps:

1. The attacker secures administrative access to a domain-joined system or escalates privileges to acquire the necessary rights to request replication data.

2. The attacker registers a rogue domain controller within the domain, leveraging the Directory Replicator permission, and executes changes to AD objects, such as modifying user groups to Domain Administrator groups.

<img width="950" height="463" alt="image" src="https://github.com/user-attachments/assets/0eaca9b9-7195-4ee0-a2db-5f25fb1f1875" />

3. The rogue domain controller initiates replication with the legitimate domain controllers, disseminating the changes throughout the domain.

<img width="922" height="452" alt="image" src="https://github.com/user-attachments/assets/daa0fdf8-a247-4659-bbf7-006ec4a6baac" />

## DCShadow Detection Opportunities

To emulate a Domain Controller, DCShadow must implement specific modifications in Active Directory:

- Add a new nTDSDSA object
- Append a global catalog ServicePrincipalName to the computer object

Event ID 4742 (Computer account was changed) logs changes related to computer objects, including ServicePrincipalName.

## Detecting DCShadow With Splunk

Now let's explore how we can identify DCShadow, using Splunk.

**Timeframe:** `earliest=1690623888 latest=1690623890`

```shellsession id="m5x8jc"
index=main earliest=1690623888 latest=1690623890 EventCode=4742 
| rex field=Message "(?P<gcspn>XX/[a-zA-Z0-9\.\-/]+)" 
| table _time, ComputerName, Security_ID, Account_Name, user, gcspn 
| search gcspn=*
```

<img width="1880" height="596" alt="image" src="https://github.com/user-attachments/assets/20443763-712a-4f77-a77f-290477bb36b6" />

## 🔑 Key Takeaway

DCSync abuses Active Directory replication permissions to extract password hashes, including those belonging to highly privileged accounts such as KRBTGT. Event ID 4662 can help detect replication-related activity by monitoring for the DS-Replication-Get-Changes property. DCShadow uses a rogue Domain Controller to make unauthorized Active Directory changes. Detection opportunities include monitoring Event ID 4742 for suspicious computer account modifications, particularly the addition of a global catalog ServicePrincipalName.
