# Detection & Analysis Stage (Part 2)

## 🔍 Why Understanding "How" Matters

> ⚠️ **Why not just rebuild the system and move on?**

Because if you don't know **how the attacker got in**, they may simply do it again.

Understanding the full attack path — including the **entry point, tools used, systems affected, and attacker activities** — allows the security team to identify and close the exact path used during the attack.

> 💡 **Key idea:** Recovery restores the system. Investigation helps prevent the same attack path from being used again.

---

## 🔄 The Investigation Loop

Incident investigations do not always follow a straight line. Instead, they follow a **repeating investigation cycle** as new information is discovered.

```text
┌──────────────────────┐
│ Create / Use IOCs    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────────────┐
│ Find New Leads &             │
│ Impacted Systems             │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│ Collect & Analyze Data       │
└──────────┬───────────────────┘
           │
           └──────────────────────┐
                                  │
                                  ▼
                         Create / Use IOCs


```

<img width="1138" height="658" alt="image" src="https://github.com/user-attachments/assets/dc2448db-3564-4d83-9312-0ff3595275dd" />


The cycle continues until investigators have a sufficiently complete understanding of the incident.

> ⚠️ **Avoid tunnel vision:** Do not focus only on one finding, such as a known malware tool. Narrowing the investigation too early can result in missed systems, incorrect conclusions, and an incomplete understanding of the incident.

---

## 1. 🧩 Indicators of Compromise (IOCs)

An **Indicator of Compromise (IOC)** is evidence that indicates potentially malicious activity has occurred.

Common IOC types include:

* File hashes — MD5, SHA-256
* IP addresses
* Domain names
* File names
* Registry keys

### 📋 Common IOC Formats and Standards

| Format / Standard | What It Is                                                                 |
| ----------------- | -------------------------------------------------------------------------- |
| **OpenIOC**       | A language for writing and sharing IOCs in a structured format.            |
| **YARA**          | Rules used to identify malware based on patterns and characteristics.      |
| **STIX**          | A JSON-based format used to represent and share cyber threat intelligence. |

> 💡 **Example:** YARA rules can be particularly useful during malware analysis because they can identify files based on specific characteristics or patterns rather than relying only on file names or hashes.

### 🛠️ IOC Tools

**Mandiant IOC Editor** is a tool that can be used to create and edit IOCs.

IOCs can also be obtained from **external threat-intelligence sources**. When a particular attacker or malware family is already known, organizations such as CISA may publish threat-intelligence information containing reusable indicators.

---

## 2. 📄 Reading a STIX Example

A STIX entry can contain information that helps investigators identify a suspicious file across an environment.

For example:

```...SNIP...
        {
            "type": "file",
            "spec_version": "2.1",
            "id": "file--474454e8-d393-5a4f-9069-19631ea9d397",
            "hashes": {
                "MD5": "40e609840ef3f7fea94d53998ec9f97f",
                "SHA-1": "141af6bcefdcf6b627425b5b2e02342c081e8d36",
                "SHA-256": "3461da3a2ddcced4a00f87dcd7650af48f97998a3ac9ca649d7ef3b7332bd997",
                "SHA-512": "deaed6b7657cc17261ae72ebc0459f8a558baf7b724df04d8821c7a5355e037a05c991433e48d36a5967ae002459358678873240e252cdea4dcbcd89218ce5c2",
                "SSDEEP": "384:cMQLQ5VU1DcZugg2YBAxeFMxeFAReF9ReFj4U0QiKy8Mg3AxeFaxeFAReFLxTYma:ElHh1gtX10u5A"
            },
            "size": 13373,
            "name": "osvmhdfl.dll",
            "object_marking_refs": [
                "marking-definition--94868c89-83c2-464b-929b-a1a8aa3c8487",
                "marking-definition--d896763f-3f6f-4917-86e8-1a4b043d9771"
            ],
            "extensions": {
                "windows-pebinary-ext": {
                    "pe_type": "dll",
                    "number_of_sections": 4,
                    "time_date_stamp": "2025-07-22T08:33:22Z",
                    "size_of_optional_header": 512,
                    "sections": [
                        {
                            "name": "header",
                            "size": 512,
                            "entropy": 2.545281,
                            "hashes": {
                                "MD5": "2a11da5809d47c180a7aa559605259b5"
                            }
                        },
                        {
                            "name": ".text",
                            "size": 4608,
                            "entropy": 4.532967,
                            "hashes": {
                                "MD5": "531ff1038e010be3c55de9cf1f212b56"
                            }
                        },
                        {
                            "name": ".rsrc",
                            "size": 1024,
                            "entropy": 2.170401,
                            "hashes": {
                                "MD5": "ef6793ef1a2f938cddc65b439e44ea07"
                            }
                        },
                        {
                            "name": ".reloc",
                            "size": 512,
                            "entropy": 0.057257,
                            "hashes": {
                                "MD5": "403090c0870bb56c921d82a159dca5a3"
                            }
                        }
                    ]
                }
            }
        },
...SNIP...
```

### 🔎 What the Fields Tell You

| Field             | Meaning                                                                      |
| ----------------- | ---------------------------------------------------------------------------- |
| `name`            | The file name to search for.                                                 |
| `MD5`             | An MD5 hash that can be used to identify the specific file.                  |
| `SHA-256`         | A stronger cryptographic hash used for file identification and verification. |
| `size`            | The file size in bytes.                                                      |
| `pe_type`         | Indicates that the file is a Windows PE file and, in this example, a DLL.    |
| `time_date_stamp` | The recorded compilation timestamp of the PE file.                           |

### 📊 File Entropy

**Entropy** measures how random the data within a file or section appears to be.

For example:

```text
Entropy: 4.532967
```

Higher entropy in certain executable sections can indicate that the code is **packed or encrypted**.

> ⚠️ **Important:** High entropy alone does **not** prove that a file is malicious. It is an indicator that should be investigated alongside other evidence.

### 🧪 Practical Investigation

In an actual investigation, analysts can take the hashes from the IOC and search across the environment:

```text
Does this file exist anywhere in our environment?
```

For example:

```text
SHA-256
   ↓
Search EDR / SIEM / File Inventory
   ↓
Identify matching systems
   ↓
Investigate each match
```

---

## 3. 🔎 Find New Leads and Impacted Systems

Once investigators have identified useful IOCs, they can search for those indicators across the organization.

The goal is to identify:

* Other machines containing the same malicious file
* Other systems communicating with the same malicious IP address
* Other accounts showing similar suspicious behavior
* Additional files or domains associated with the attack

Every new match can become a **new investigation lead**.

```text
IOC
 │
 ▼
Search Environment
 │
 ├── Host A → Match
 ├── Host B → Match
 ├── Host C → No Match
 └── Host D → Match
       │
       ▼
 New Investigation Leads
```

> 💡 **Key idea:** Every validated new finding should be added to the incident timeline and used to continue the investigation loop.

---

## 4. 🧪 Data Collection and Analysis

When an impacted system is identified, investigators need to collect information about its current and historical state.

There are two common approaches.

### 🟢 Live Response

Live response collects evidence while the system is still running.

It can capture volatile information such as:

* RAM / memory
* Running processes
* Active network connections
* Logged-in users
* Open handles and sessions
* Other runtime artifacts

> ⚠️ **Important:** Live systems are changing continuously. The more investigators interact with the system, the greater the possibility of altering evidence.

For this reason, investigators should minimize unnecessary interaction and follow established incident-response procedures.

### 🔴 Shut Down Then Analyze

In some situations, investigators may shut down the affected system before performing further analysis.

However:

> ⚠️ **Critical:** Data that exists only in RAM is lost when the system is powered off.

This creates an important trade-off between preserving volatile evidence and containing the incident.

### 🧠 Analysis Types

After collecting evidence, investigators may perform several types of analysis:

| Analysis Type        | Purpose                                                                                                        |
| -------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Malware Analysis** | Examines malicious files to understand their behavior, capabilities, and characteristics.                      |
| **Disk Forensics**   | Examines files, logs, and other artifacts written to storage.                                                  |
| **Memory Forensics** | Examines RAM captures to identify processes, network connections, injected code, and other volatile artifacts. |

> 📌 **Good practice:** Add validated findings to the **incident timeline** throughout the investigation rather than waiting until the end.

---

## 5. ⚖️ Chain of Custody

Every piece of evidence collected during an investigation should have a **documented chain of custody**.

A chain of custody records information such as:

* Who collected the evidence
* When it was collected
* Where it was collected
* How it was collected
* Who handled the evidence afterward
* When the evidence was transferred

```text
Evidence Collected
       ↓
Document Collection
       ↓
Evidence Stored
       ↓
Evidence Transferred
       ↓
Evidence Analyzed
```

> ⚠️ **Important:** Proper evidence handling becomes especially important when an incident may result in legal or disciplinary proceedings. Poor documentation can create questions about the integrity and handling of the evidence.

---

# 🤖 AI in Threat Detection and Incident Response

AI is increasingly being used to help SOC teams handle large volumes of security alerts.

### Traditional Approach

```text
Thousands of Alerts
       ↓
Manual Review
       ↓
Correlation
       ↓
Investigation
       ↓
Incident Story
```

This can require significant analyst time.

### AI-Assisted Approach

```text
Thousands of Alerts
       ↓
AI Analysis
       ↓
Alert Correlation
       ↓
Behavioral Patterns
       ↓
Related Alerts Grouped
       ↓
Attack Story
```

> 💡 **Key idea:** AI can help reduce the amount of repetitive analysis required from security analysts.

<img width="1238" height="750" alt="image" src="https://github.com/user-attachments/assets/a20c8ea3-4590-476a-825d-c9abac4d7549" />

---

## 🛡️ Example: Elastic Security Attack Discovery

One example of AI-assisted security analysis is **Elastic Security's Attack Discovery** capability.

It uses a Large Language Model (LLM) to help analyze security alerts and identify relationships between them.

Depending on the available telemetry and configuration, this type of capability can help:

* Group related alerts
* Identify users and hosts involved in an attack
* Build a more coherent attack story
* Map activity to MITRE ATT&CK techniques

---

## 📋 Common AI Use Cases in Incident Response

| Use Case                                           | What It Does                                                                                                |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Automated Triage & Prioritization**              | Helps prioritize alerts so analysts can focus on potentially important activity.                            |
| **Incident Correlation & Timeline Reconstruction** | Links related alerts across systems to create a more coherent incident timeline.                            |
| **Automated Response Playbooks**                   | Can trigger predefined response actions when specific conditions are detected.                              |
| **Post-Incident Analysis**                         | Helps identify recurring patterns across previous incidents and support improvements to defensive controls. |

---

## 🧠 AI Does Not Replace the Analyst

> 💡 **Key takeaway:** AI is primarily useful for helping analysts manage the **volume and complexity** of security data.

```text
AI
 │
 ├── Analyze large volumes of data
 ├── Correlate related alerts
 ├── Identify patterns
 └── Automate repetitive tasks
          │
          ▼
     SOC Analyst
          │
          ├── Validate findings
          ├── Investigate context
          ├── Make decisions
          └── Take appropriate action
```

Human analysts remain responsible for validating findings, understanding context, making investigation decisions, and determining appropriate response actions.

---

## ✅ Key Takeaways

1. **Understand how the attacker entered and operated**, not just how to recover the affected system.
2. Use **IOCs** to identify malicious artifacts and search for additional affected systems.
3. Treat the investigation as a **continuous loop** rather than a one-time process.
4. Collect volatile evidence carefully because **RAM data can disappear when a system is powered off**.
5. Use malware, disk, and memory analysis to build a complete understanding of the incident.
6. Maintain a proper **chain of custody** when collecting and handling evidence.
7. Use AI to help manage alert volume, correlate events, and identify patterns.
8. **Human validation and judgment remain important** throughout the investigation.
