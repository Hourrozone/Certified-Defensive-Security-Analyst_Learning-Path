# 🧪 Skills Assessment

In this skills assessment, you will work as a **Junior Incident Responder**.

You will practice several real incident-response tasks using tools such as **TheHive**, **MITRE ATT&CK**, and collected security logs.

The assessment covers three main areas:

```text
🧪 Skills Assessment
        │
        ├── 🛡️ Triage the Alerts
        │
        ├── 🔗 Map the Attack
        │
        └── 🔍 Investigate the Logs
```

> 🧠 **Junior Incident Responder:** A security team member who helps monitor, investigate, and respond to security incidents.

---

# 🛡️ 1. Triage the Alerts

**Triage** means quickly reviewing security alerts to understand:

* What happened
* How serious the alert may be
* Which alerts are related
* What should be investigated first

In this exercise, **TheHive** contains several alerts related to the **Insights Nexus breach**.

Your task is to identify the alerts related to this incident and investigate them.

---

## 📌 Task 1 — Create a Case in TheHive

Create a new **case** in TheHive.

First, find all alerts that are related to the **Insights Nexus breach scenario**.

Then:

1. Create a new case.
2. Find the related alerts.
3. Link those alerts to the case.

```text
TheHive Alerts
      │
      ├── Alert A ──┐
      ├── Alert B ──┼──→ Insights Nexus Case
      ├── Alert C ──┤
      └── Alert D ──┘
```
<img width="1590" height="751" alt="image" src="https://github.com/user-attachments/assets/9c6d0104-04f4-4f6a-b780-20892e628c26" />

> 💡 **Why is this important?** A case groups related alerts together so analysts can investigate the entire incident in one place.

This task introduces you to working with **alerts and cases in TheHive**.

---

## 🔎 Task 2 — Triage, Enrichment, and Correlation

Next, perform three important investigation activities:

### Triage

Review the alert and determine what information is important.

### Enrichment

Add additional information that helps explain the alert.

For example, you may add information about:

* IP addresses
* Domains
* File hashes
* Malware
* Users
* Hosts

Useful information can be added to the **notes** of an alert.

> 🧠 **Enrichment:** Adding more information to an alert so that the analyst can better understand and investigate it.

### Correlation

Look for relationships between different alerts, users, hosts, IP addresses, or other indicators.

```text
Alert A
  │
  ├── Same IP ──→ Alert B
  │
  ├── Same Host ─→ Alert C
  │
  └── Same User ─→ Alert D
```

> 💡 **Key idea:** Correlation helps analysts see that several alerts may actually be part of the **same attack**.

---

## 🌐 Task 3 — Investigate External Connections

One of the Insights Nexus alerts contains information in its **notes**.

The notes include output from the:

```text
netstat
```

command.

`netstat` can show network connections made by a computer.

The output shows that the host is communicating with some **external IP addresses**.

Your task is to validate this finding.

### 🔍 Investigation Scenario

The `netstat` output was collected after the host machine had **rejoined the domain following recovery**.

However, the computer was still communicating with an external IP address.

The analyst added this information to the alert comments.

You need to investigate the connection and determine what it means.

```text
Recovered Host
      │
      ↓
Rejoined Domain
      │
      ↓
netstat Output
      │
      ↓
External IP Connection
      │
      ↓
🔍 Validate the Finding
```

> ⚠️ **Important:** An external connection does not automatically mean that the computer is compromised. You need to investigate the IP address and the context of the connection before reaching a conclusion.

There are additional questions at the bottom of this section that you need to answer based on your investigation.

---

# 🔗 2. Mapping the Attack to the Cyber Kill Chain

In this section, you will analyze a complete attack scenario.

The attack begins when a user opens a malicious attachment.

The attachment executes a **downloader**, which performs several actions:

```text
User opens attachment
        ↓
Downloader executes
        ↓
Writes .exe file to %AppData%
        ↓
Creates a Run registry key
        ↓
Loads VaultCli.dll
        ↓
Credentials are accessed
        ↓
Credentials are sent to an external IP
```

Your task is to understand each step and map it to the appropriate attack framework.

---

## 🧠 What Is the Cyber Kill Chain?

The **Cyber Kill Chain** is a model used to describe the different stages of a cyber attack.

It helps analysts understand how an attack progresses from the beginning to the final objective.

```text
Attack Begins
     ↓
Reconnaissance
     ↓
Weaponization
     ↓
Delivery
     ↓
Exploitation
     ↓
Installation
     ↓
Command & Control
     ↓
Actions on Objectives
```

> 💡 **Key idea:** Mapping attacker activity to the Cyber Kill Chain helps analysts understand **where each action fits in the overall attack**.

---

## 📌 Task 1 — Map the Activity to MITRE ATT&CK

Identify the MITRE ATT&CK techniques related to:

* File download
* Registry modification
* Credential activity
* Data exfiltration

For example:

```text
File Download
     ↓
Identify the MITRE ATT&CK Technique
```

```text
Registry Modification
     ↓
Identify the MITRE ATT&CK Technique
```

```text
Data Exfiltration
     ↓
Identify the MITRE ATT&CK Technique
```

> 🧠 **MITRE ATT&CK:** A knowledge base that documents common techniques and tactics used by attackers.

---

## 🔎 Task 2 — Investigate the Mimikatz Alert

In TheHive, find the alert related to **Mimikatz**.

Review the alert information and identify the **MITRE ATT&CK Technique ID** associated with the activity.

Your task is to:

1. Find the Mimikatz alert.
2. Review the information.
3. Identify the associated MITRE ATT&CK technique.
4. Record the **Technique ID**.

> 📌 **Tip:** Do not rely only on the tool name. Look at what the tool is being used to do and identify the technique associated with that behavior.

---

# 🔍 3. Investigate the Collected Logs

In this section, you will work with collected security event logs.

You are provided with a file:

```text
logs-wazuh.zip
```

The logs contain information that can help you investigate suspicious activity.

---

## 💻 Task 1 — Decode PowerShell Commands and Find IOCs

Some PowerShell commands in the logs are encoded.

Your task is to:

1. Find the suspicious PowerShell command.
2. Decode the command.
3. Understand what the command does.
4. Extract any **IOCs** from the decoded command.

Possible IOCs may include:

* IP addresses
* Domain names
* URLs
* File names
* File hashes
* File paths

The investigation process can look like:

```text
Wazuh Logs
    ↓
Find Suspicious PowerShell
    ↓
Decode Command
    ↓
Understand the Command
    ↓
Extract IOCs
    ↓
Investigate the IOCs
```

> ⚠️ **Important:** Do not execute a suspicious PowerShell command just to understand it. Decode and analyze it safely in an appropriate analysis environment.

---

## 👤 Task 2 — Identify the User

Finally, identify the **user account that executed the suspicious PowerShell command**.

Look for information in the event logs such as:

* Username
* Account name
* User ID
* Process information
* PowerShell execution events

The goal is to connect the suspicious command to the account that executed it.

```text
Suspicious PowerShell Event
          ↓
Process Information
          ↓
User Information
          ↓
👤 Identify the User
```

> 🔍 **Investigation tip:** Do not assume that the logged-in user is automatically the person who executed the command. Use the event information to verify the account associated with the PowerShell process.

---

# 🧭 Assessment Workflow

The entire assessment can be summarized as:

```text
┌─────────────────────────┐
│   🛡️ Triage Alerts      │
│                         │
│ TheHive                 │
│ Cases + Alerts          │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   🔗 Map the Attack     │
│                         │
│ Kill Chain +            │
│ MITRE ATT&CK            │
└────────────┬────────────┘
             ↓
┌─────────────────────────┐
│   🔍 Investigate Logs   │
│                         │
│ PowerShell + IOCs +     │
│ User Identification     │
└─────────────────────────┘
```

---

# 🧠 Beginner Summary

This skills assessment is testing whether you can perform basic incident-response activities.

You should be able to:

1. **Triage alerts** in TheHive.
2. Create a **case** and link related alerts.
3. Add information to alerts through **enrichment**.
4. Find relationships between alerts through **correlation**.
5. Investigate suspicious **network connections**.
6. Understand the **Cyber Kill Chain**.
7. Map attacker behavior to **MITRE ATT&CK**.
8. Investigate a **Mimikatz** alert.
9. Analyze collected **Wazuh logs**.
10. Safely decode suspicious **PowerShell commands**.
11. Extract **IOCs** from suspicious commands.
12. Identify the **user account** associated with suspicious activity.

> ✅ **Key takeaway:** The goal is not just to find suspicious activity. You should be able to **connect the evidence, understand what happened, identify the attacker behavior, and document your findings**.
