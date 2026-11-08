# Detecting Common User/Domain Recon

## Domain Reconnaissance

Active Directory (AD) domain reconnaissance represents a pivotal stage in the cyberattack lifecycle. During this phase, adversaries endeavor to gather information about the target environment, seeking to comprehend its architecture, network topology, security measures, and potential vulnerabilities.

While conducting AD domain reconnaissance, attackers focus on identifying crucial components such as Domain Controllers, user accounts, groups, trust relationships, organizational units (OUs), group policies, and other vital objects. By gaining insights into the AD environment, attackers can potentially pinpoint high-value targets, escalate their privileges, and move laterally within the network.

## User/Domain Reconnaissance Using Native Windows Executables

An example of AD domain reconnaissance is when an adversary executes the `net group` command to obtain a list of Domain Administrators.

<img width="659" height="232" alt="image" src="https://github.com/user-attachments/assets/d6cfaee3-91f0-479a-b718-373e40c11812" />

Common native tools/commands utilized for domain reconnaissance include:

```cmd
whoami /all
wmic computersystem get domain
net user /domain
net group "Domain Admins" /domain
arp -a
nltest /domain_trusts
```

For detection, administrators can employ PowerShell to monitor for unusual scripts or cmdlets and process command-line monitoring.

## User/Domain Reconnaissance Using BloodHound/SharpHound

BloodHound is an open-source domain reconnaissance tool created to analyze and visualize the Active Directory (AD) environment. It is frequently employed by attackers to discern attack paths and potential security risks within an organization's AD infrastructure. BloodHound leverages graph theory and relationship mapping to elucidate trust relationships, permissions, and group memberships within the AD domain.

<img width="1814" height="1230" alt="image" src="https://github.com/user-attachments/assets/f9e71c10-9184-400c-80d4-04814c7c7184" />

Sharphound is a C# data collector for BloodHound. An example of usage includes an adversary running Sharphound with all collection methods (`-c all`).

<img width="831" height="372" alt="image" src="https://github.com/user-attachments/assets/cfac3170-1f7f-4aac-aa03-f446eb6d33f3" />

### BloodHound Detection Opportunities

Under the hood, the BloodHound collector executes numerous LDAP queries directed at the Domain Controller, aiming to amass information about the domain.

<img width="1057" height="623" alt="image" src="https://github.com/user-attachments/assets/ffd6e3ef-713c-4ab9-8166-08c14eeb7108" />

However, monitoring LDAP queries can be a challenge. By default, the Windows Event Log does not record them. The best option Windows can suggest is employing Event 1644 - the LDAP performance monitoring log. Even with it enabled, BloodHound may not generate many of the expected events.

<img width="568" height="278" alt="image" src="https://github.com/user-attachments/assets/03fa2c14-9a84-4221-96d0-7b8e3fe3e2ef" />

A more reliable approach is to utilize the Windows ETW provider `Microsoft-Windows-LDAP-Client`. As showcased previously in the SOC Analyst path, SilkETW & SilkService are versatile C# wrappers for ETW, designed to simplify the intricacies of ETW, providing an accessible interface for research and introspection. SilkService supports output to the Windows Event Log, which streamlines log digestion. Another useful feature is the ability to employ Yara rules for hunting suspicious LDAP queries.

<img width="1048" height="760" alt="image" src="https://github.com/user-attachments/assets/9391b3f9-ce21-49ec-878a-ce5635d10352" />

In addition, Microsoft's ATP team has compiled a list of LDAP filters frequently used by reconnaissance tools.

<img width="650" height="757" alt="image" src="https://github.com/user-attachments/assets/c5fbb564-6a89-4fbc-8ef6-3aabc41d0f6f" />

Armed with this list of LDAP filters, BloodHound activity can be detected more efficiently.

---

## Detecting User/Domain Recon With Splunk

You'll observe that a specific timeframe is given when identifying each attack. This is done to concentrate on the relevant events, avoiding the overwhelming volume of unrelated events.

Now let's explore how we can identify the recon techniques previously discussed, using Splunk.

### Detecting Recon By Targeting Native Windows Executables

**Timeframe:** `earliest=1690447949 latest=1690450687`

<img width="736" height="202" alt="image" src="https://github.com/user-attachments/assets/0d4ebe64-1a34-496f-8c62-0d418ee6076a" />

<img width="1884" height="1037" alt="image" src="https://github.com/user-attachments/assets/c8ea5ed7-2452-451c-a74a-20bb159ab76c" />

#### Search Breakdown

1. **Filtering by Index and Source:** The search begins by selecting events from the main index where the source is `XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`, which is the XML-formatted Windows Event Log for Sysmon (System Monitor) events. Sysmon is a service and device driver that logs system activity to the event log.

2. **EventID Filter:** The search is further filtered to only select events with an Event ID of 1. In Sysmon, Event ID 1 corresponds to Process Creation events, which log data about newly created processes.

3. **Time Range Filter:** The search restricts the time range of events to those occurring between the Unix timestamps `1690447949` and `1690450687`. These timestamps represent the earliest and latest times in which the events occurred.

4. **Process Name Filter:** The search then filters events to only include those where the `process_name` field is one of a list of specific process names (e.g., `arp.exe`, `chcp.com`, `ipconfig.exe`, etc.) or where the `process_name` field is `cmd.exe` or `powershell.exe` and the `process` field contains certain substrings. This step is looking for events that involve certain system or network-related commands, as well as events where these commands were run from a Command Prompt or PowerShell session.

5. **Statistics:** The `stats` command is used to aggregate events based on the fields `parent_process`, `parent_process_id`, `dest`, and `user`. For each unique combination of these fields, the search calculates the following statistics:
   - `values(process) as process`: This captures all unique values of the `process` field as a multivalue field named `process`.
   - `min(_time) as _time`: This captures the earliest time (`_time`) that an event occurred within each group.

6. **Filtering by Process Count:** The `where` command is used to filter the results to only include those where the count of the `process` field is greater than 3. This step is looking for instances where multiple processes (more than three) were executed by the same parent process.

### Detecting Recon By Targeting BloodHound

**Timeframe:** `earliest=1690195896 latest=1690285475`

<img width="730" height="212" alt="image" src="https://github.com/user-attachments/assets/98b38ec1-e10f-421b-9664-77c6c1a94181" />

<img width="1882" height="868" alt="image" src="https://github.com/user-attachments/assets/f5a07453-92b2-4116-a0ac-18edf5f3a583" />

#### Search Breakdown

1. **Filtering by Index and Source:** The search starts by selecting events from the main index where the source is `WinEventLog:SilkService-Log`. This source represents Windows Event Log data gathered by SilkETW.

2. **Time Range Filter:** The search restricts the time range of events to those occurring between the Unix timestamps `1690195896` and `1690285475`. These timestamps represent the earliest and latest times in which the events occurred.

3. **Path Extraction:** The `spath` command is used to extract fields from the `Message` field, which likely contains structured data such as XML or JSON. The `spath` command automatically identifies and extracts fields based on the data structure.

4. **Field Renaming:** The `rename` command is used to rename fields that start with `XmlEventData.` to the equivalent field names without the `XmlEventData.` prefix. This is done for easier reference to the fields in later stages of the search.

5. **Tabulating Results:** The `table` command is used to display the results in a tabular format with the following columns: `_time`, `ComputerName`, `ProcessName`, `ProcessId`, `DistinguishedName`, and `SearchFilter`. The `table` command only includes these fields in the output.

6. **Sorting:** The `sort` command is used to sort the results based on the `_time` field in ascending order (from oldest to newest). The `0` argument means that there is no limit on the number of results to sort.

7. **Search Filter:** The `search` command is used to filter the results to only include events where the `SearchFilter` field contains the string `*(samAccountType=805306368)*`. This step is looking for events related to LDAP queries with a specific filter condition.

8. **Statistics:** The `stats` command is used to aggregate events based on the fields `ComputerName`, `ProcessName`, and `ProcessId`. For each unique combination of these fields, the search calculates the following statistics:
   - `min(_time) as _time`: The earliest time (`_time`) that an event occurred within each group.
   - `max(_time) as maxTime`: The latest time (`_time`) that an event occurred within each group.
   - `count`: The number of events within each group.
   - `values(SearchFilter) as SearchFilter`: All unique values of the `SearchFilter` field within each group.

9. **Filtering by Event Count:** The `where` command is used to filter the results to only include those where the `count` field is greater than 10. This step is looking for instances where the same process on the same computer made more than ten search queries with the specified filter condition.

10. **Time Conversion:** The `convert` command is used to convert the `maxTime` field from Unix timestamp format to human-readable format (`ctime`).

---

## 🔑 Key Takeaway

Active Directory reconnaissance involves gathering information about users, groups, domain controllers, trust relationships, and other domain objects. Attackers may use native Windows commands or tools such as BloodHound and SharpHound. Detection relies on monitoring suspicious process execution, command-line activity, LDAP queries, and ETW telemetry. Splunk can correlate these events to identify unusual reconnaissance patterns, including repeated LDAP queries associated with BloodHound.
