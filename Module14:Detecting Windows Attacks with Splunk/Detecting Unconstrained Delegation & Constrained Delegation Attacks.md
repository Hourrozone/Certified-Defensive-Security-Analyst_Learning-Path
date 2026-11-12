<img width="484" height="345" alt="image" src="https://github.com/user-attachments/assets/f5b5161e-637a-40e1-8337-b3fe3ae82d77" /># Detecting Unconstrained Delegation & Constrained Delegation Attacks

## Unconstrained Delegation

Unconstrained Delegation is a privilege that can be granted to User Accounts or Computer Accounts in an Active Directory environment, allowing a service to authenticate to another resource on behalf of any user. This might be necessary when, for example, a web server requires access to a database server to make changes on a user's behalf.

<img width="462" height="528" alt="image" src="https://github.com/user-attachments/assets/65ba87d8-73f6-4138-9b78-a16acd0a177f" />

### Attack Steps:

1. The attacker identifies systems on which Unconstrained Delegation is enabled for service accounts.

<img width="856" height="362" alt="image" src="https://github.com/user-attachments/assets/727617bd-88e8-4cb6-806f-c73a520e0c64" />

2. The attacker gains access to a system with Unconstrained Delegation enabled.

3. The attacker extracts Ticket Granting Ticket (TGT) tickets from the memory of the compromised system using tools such as Mimikatz.

<img width="882" height="376" alt="image" src="https://github.com/user-attachments/assets/1fe33366-b01f-441e-9fbf-88848d89447f" />

## Kerberos Authentication With Unconstrained Delegation

When Unconstrained Delegation is enabled, the main difference in Kerberos Authentication is that when a user requests a TGS ticket for a remote service, the Domain Controller will embed the user's TGT into the service ticket. When connecting to the remote service, the user will present not only the TGS ticket but also their own TGT. When the service needs to authenticate to another service on behalf of the user, it will present the user's TGT ticket, which the service received with the TGS ticket.

<img width="484" height="345" alt="image" src="https://github.com/user-attachments/assets/6120c8ea-d83f-40fc-a5e2-739034fd69e4" />

## Unconstrained Delegation Attack Detection Opportunities

PowerShell commands and LDAP search filters used for Unconstrained Delegation discovery can be detected by monitoring PowerShell script block logging (Event ID 4104) and LDAP request logging.

The main goal of an Unconstrained Delegation attack is to retrieve and reuse TGT tickets, so Pass-the-Ticket detection can be used as well.

<img width="1672" height="583" alt="image" src="https://github.com/user-attachments/assets/95753c1f-102a-4caa-83fd-b4ed4b227719" />

## Detecting Unconstrained Delegation Attacks With Splunk

Now let's explore how we can identify Unconstrained Delegation attacks, using Splunk.

**Timeframe:** `earliest=1690544538 latest=1690544540`

```shellsession
index=main earliest=1690544538 latest=1690544540 source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*TrustedForDelegation*" OR Message="*userAccountControl:1.2.840.113556.1.4.803:=524288*" 
| table _time, ComputerName, EventCode, Message
```

<img width="406" height="341" alt="image" src="https://github.com/user-attachments/assets/ca6b696f-03d5-4250-a720-a3bd419b3284" />

## Constrained Delegation

Constrained Delegation is a feature in Active Directory that allows services to delegate user credentials only to specified resources, reducing the risk associated with Unconstrained Delegation. Any user or computer accounts that have service principal names (SPNs) set in their msDS-AllowedToDelegateTo property can impersonate any user in the domain to those specific SPNs.

<img width="856" height="336" alt="image" src="https://github.com/user-attachments/assets/682c97a4-796a-4971-9709-a69680d3ba71" />

### Attack Steps:

1. The attacker identifies systems where Constrained Delegation is enabled and determines the resources to which they are allowed to delegate.

<img width="976" height="437" alt="image" src="https://github.com/user-attachments/assets/b4e4df70-653f-47cc-b789-cc1906d7c9ce" />

2. The attacker gains access to the TGT of the principal (user or computer). The TGT can be extracted from memory (Rubeus dump) or requested with the principal's hash.

<img width="977" height="529" alt="image" src="https://github.com/user-attachments/assets/d882d0f2-8c1a-460a-86f8-c1863fae1505" />

3. The attacker uses the S4U technique to impersonate a high-privileged account to the targeted service (requesting a TGS ticket).

<img width="977" height="529" alt="image" src="https://github.com/user-attachments/assets/aedc977b-4085-433d-a115-a16fea1a68f4" />

4. The attacker injects the requested ticket and accesses targeted services as the impersonated user.

<img width="974" height="577" alt="image" src="https://github.com/user-attachments/assets/0909dae7-4e44-4800-ac59-9d650332f78d" />

## Kerberos Protocol Extensions - Service For User

Service for User to Self (S4U2self) and Service for User to Proxy (S4U2proxy) allow a service to request a ticket from the Key Distribution Center (KDC) on behalf of a user. S4U2self allows a service to obtain a TGS for itself on behalf of a user, while S4U2proxy allows the service to obtain a TGS on behalf of a user for a second service.

S4U2self was designed to enable a user to request a TGS ticket when another method of authentication was used instead of Kerberos. Importantly, this TGS ticket can be requested on behalf of any user, for example, an Administrator.

<img width="487" height="423" alt="image" src="https://github.com/user-attachments/assets/c9f74543-5390-411e-ab3b-a39f676b8ceb" />

S4U2proxy was designed to take a forwardable ticket and use it to request a TGS ticket to any SPN specified in the msds-allowedtodelegateto options for the user specified in the S4U2self part.

With a combination of S4U2self and S4U2proxy, an attacker can impersonate any user to service principal names (SPNs) set in msDS-AllowedToDelegateTo properties.

## Constrained Delegation Attack Detection Opportunities

Similar to Unconstrained Delegation, it is possible to detect PowerShell commands and LDAP requests aimed at discovering vulnerable Constrained Delegation users and computers.

To request a TGT ticket for a principal, as well as a TGS ticket using the S4U technique, Rubeus makes connections to the Domain Controller. This activity can be detected as an unusual process network connection to TCP/UDP port 88 (Kerberos).

## Detecting Constrained Delegation Attacks With Splunk

Now let's explore how we can identify Constrained Delegation attacks, using Splunk.

### Detecting Constrained Delegation Attacks - Leveraging PowerShell Logs

**Timeframe:** `earliest=1690544553 latest=1690562556`

```shellsession
index=main earliest=1690544553 latest=1690562556 source="WinEventLog:Microsoft-Windows-PowerShell/Operational" EventCode=4104 Message="*msDS-AllowedToDelegateTo*" 
| table _time, ComputerName, EventCode, Message
```

<img width="1668" height="742" alt="image" src="https://github.com/user-attachments/assets/a1d76cbf-c2c9-4603-9ddc-54023240a53f" />

### Detecting Constrained Delegation Attacks - Leveraging Sysmon Logs

**Timeframe:** `earliest=1690562367 latest=1690562556`

```shellsession
index=main earliest=1690562367 latest=1690562556 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" 
| eventstats values(process) as process by process_id
| where EventCode=3 AND dest_port=88
| table _time, Computer, dest_ip, dest_port, Image, process
```

<img width="1907" height="1239" alt="image" src="https://github.com/user-attachments/assets/a3092e6d-31ad-40e4-b2a2-56b161a585dc" />

## 🔑 Key Takeaway

Unconstrained Delegation allows services to receive users' TGTs, which attackers can extract and reuse for lateral movement. Constrained Delegation limits delegation to specified SPNs, but attackers can abuse S4U2self and S4U2proxy to impersonate privileged users to those services. Detection opportunities include monitoring PowerShell script block logs (Event ID 4104), LDAP requests, Pass-the-Ticket activity, and unusual process connections to Domain Controllers over TCP/UDP port 88.
