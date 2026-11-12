<img width="1668" height="468" alt="image" src="https://github.com/user-attachments/assets/9029c066-2bf5-4d84-a7f3-e1c721d7c4e9" /><img width="1645" height="724" alt="image" src="https://github.com/user-attachments/assets/7a514b54-670b-431b-8390-466f50bb0e2c" /># Detecting Pass-the-Hash

## Pass-the-Hash

Pass-the-Hash is a technique utilized by attackers to authenticate to a networked system using the NTLM hash of a user's password instead of the plaintext password. The attack capitalizes on the way Windows stores password hashes in memory, enabling adversaries with administrative access to capture the hash and reuse it for lateral movement within the network.

### Attack Steps:

1. The attacker employs tools such as Mimikatz to extract the NTLM hash of a user currently logged onto the compromised system. Note that local administrator privileges are required on the system to extract the user's hash.

<img width="962" height="476" alt="image" src="https://github.com/user-attachments/assets/76379c3f-f47b-4999-9c62-b2a7a9e694e8" />

2. Armed with the NTLM hash, the attacker can authenticate as the targeted user on other systems or network resources without needing to know the actual password.

<img width="977" height="513" alt="image" src="https://github.com/user-attachments/assets/98f02654-d9be-40cf-a2c6-0f8d76cc22c1" />

3. Utilizing the authenticated session, the attacker can move laterally within the network, gaining unauthorized access to other systems and resources.

<img width="589" height="333" alt="image" src="https://github.com/user-attachments/assets/bead4617-4728-4fbf-8cbb-7bdec3f0f6d4" />

## Windows Access Tokens & Alternate Credentials

An access token is a data structure that defines the security context of a process or thread. It contains information about the associated user account's identity and privileges. When a user logs on, the system verifies the user's password by comparing it with information stored in a security database. If the password is authenticated, the system generates an access token. Subsequently, any process executed on behalf of that user possesses a copy of this access token. (Source: https://learn.microsoft.com/en-us/windows/win32/secauthz/access-tokens)

Alternate Credentials provide a way to supply different login credentials (username and password) for specific actions or processes without altering the user's primary login session. This permits a user or process to execute certain commands or access resources as a different user without logging out or switching user accounts. The `runas` command is a Windows command-line tool that allows users to execute commands as another user. When the `runas` command is executed, a new access token is generated, which can be verified with the `whoami` command.

<img width="849" height="599" alt="image" src="https://github.com/user-attachments/assets/fe5c6bcb-28c9-4cc4-87cf-251e51968689" />

The `runas` command also contains an interesting flag `/netonly`. This flag indicates that the specified user information is for remote access only. Even though the `whoami` command returns the original username, the spawned `cmd.exe` can still access the Domain Controller root folder.

<img width="976" height="635" alt="image" src="https://github.com/user-attachments/assets/9c8fcaa2-6bda-4aca-b91d-f664b1d56265" />

Each access token references a LogonSession generated at user logon. This LogonSession security structure contains such information as Username, Domain, and AuthenticationID (NTHash/LMHash), and is used when the process attempts to access remote resources. When the netonly flag is used, the process has the same access token but a different LogonSession.

<img width="887" height="272" alt="image" src="https://github.com/user-attachments/assets/60c87333-8c1d-4769-b0d8-b56ec0e4f401" />

## Pass-the-Hash Detection Opportunities

From the Windows Event Log perspective, the following logs are generated when the `runas` command is executed:

- When runas command is executed without the `/netonly` flag - Event ID 4624 (Logon) with LogonType 2 (interactive).

<img width="1646" height="745" alt="image" src="https://github.com/user-attachments/assets/59bf8cb3-687c-4020-a0df-edbc68bdbf5d" />

- When runas command is executed with the `/netonly` flag - Event ID 4624 (Logon) with LogonType 9 (NewCredentials).

<img width="1645" height="724" alt="image" src="https://github.com/user-attachments/assets/d623dc14-f0f0-433b-888e-b0c1a2003339" />

Simple detection would involve looking for Event ID 4624 and LogonType 9, but as mentioned before, there could be some false positives related to runas usage.

The main difference between runas with the netonly flag and the Pass-the-Hash attack is that in the latter case, Mimikatz will access the LSASS process memory to change LogonSession credential materials. Thus, initial detection can be enhanced by correlating User Logon with NewCredentials events with Sysmon Process Access Event Code 10.

## Detecting Pass-the-Hash With Splunk

Now let's explore how we can identify Pass-the-Hash, using Splunk.

Before we move on to reviewing the searches, please consult this source to gain a better understanding of where the search part `Logon_Process=seclogo` originated from.

**Timeframe:** `earliest=1690450689 latest=1690451116`

```shellsession
index=main earliest=1690450708 latest=1690451116 source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo
| table _time, ComputerName, EventCode, user, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
```

<img width="1668" height="468" alt="image" src="https://github.com/user-attachments/assets/6a771a8f-46fc-43e2-8e8c-b566af3345f3" />

As already mentioned, we can enhance the search above by adding LSASS memory access to the mix as follows.

```shellsession
index=main earliest=1690450689 latest=1690451116 (source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe" SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe") OR (source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo)
| sort _time, RecordNumber
| transaction host maxspan=1m endswith=(EventCode=4624) startswith=(EventCode=10)
| stats count by _time, Computer, SourceImage, SourceProcessId, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process
| fields - count
```

<img width="1669" height="724" alt="image" src="https://github.com/user-attachments/assets/3ab1c89a-8098-41dd-a7bc-32a011a83b84" />

### Search Breakdown:

- **`index=main earliest=1690450689 latest=1690451116`:** Filters the search to only include events from the main index that occurred between the specified earliest and latest epoch timestamps.

- **`(source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational" EventCode=10 TargetImage="C:\\Windows\\system32\\lsass.exe" SourceImage!="C:\\ProgramData\\Microsoft\\Windows Defender\\platform\\*\\MsMpEng.exe")`:** Filters the search to only include Sysmon operational log events with an EventCode of 10 (Process Access). It further narrows down the results to events where the TargetImage is C:\Windows\system32\lsass.exe (indicating that the lsass.exe process is being accessed) and the SourceImage is not a known legitimate process from the Windows Defender directory.

- **`OR (source="WinEventLog:Security" EventCode=4624 Logon_Type=9 Logon_Process=seclogo)`:** Filters the search to also include Security event log events with an EventCode of 4624 (Logon), Logon_Type of 9 (NewCredentials), and Logon_Process of seclogo.

- **`| sort _time, RecordNumber`:** Sorts the events based on the _time field and then the RecordNumber field.

- **`| transaction host maxspan=1m endswith=(EventCode=4624) startswith=(EventCode=10)`:** Groups related events based on the host field, with a maximum time span of 1 minute between the start and end events. This command is used to associate process access events targeting lsass.exe with remote logon events.

- **`| stats count by _time, Computer, SourceImage, SourceProcessId, Network_Account_Domain, Network_Account_Name, Logon_Type, Logon_Process`:** Aggregates the events based on the specified fields, counting the number of occurrences for each combination of field values.

- **`| fields - count`:** Removes the count field from the results.

## 🔑 Key Takeaway

Pass-the-Hash allows attackers to authenticate using stolen NTLM hashes rather than plaintext passwords. Detection can begin by identifying Event ID 4624 with LogonType 9 and Logon_Process `seclogo`. Correlating these events with Sysmon Event ID 10 process access events targeting LSASS helps distinguish suspicious credential manipulation from legitimate `runas /netonly` activity.
