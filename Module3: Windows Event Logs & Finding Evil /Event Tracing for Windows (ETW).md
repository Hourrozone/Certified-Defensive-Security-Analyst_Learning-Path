# 🛡️ Event Tracing for Windows (ETW)

---

## 🧠 What Is ETW?

When we investigate security incidents, we usually depend on logs from Windows, Sysmon, antivirus, SIEM, and other security tools.

However, these logs do not always contain everything we need.

**Event Tracing for Windows (ETW)** is a Windows feature that can provide additional information about what is happening inside a Windows system.

According to Microsoft, ETW is a **high-speed tracing system built into Windows**. It can collect events from:

* Windows applications
* Windows kernel components
* Device drivers
* Network activity
* Processes
* Files
* Registry
* Other Windows components

### 💡 Simple Example

Imagine that you are investigating a suspicious program.

Your normal logs might tell you:

```text
malware.exe started
```

But ETW may provide additional information about what happened around that activity, depending on the provider being used.

So, instead of looking at only:

```text
Process started
```

you may be able to collect additional telemetry that helps answer questions such as:

```text
What happened?
Which process was involved?
What file was accessed?
What network activity occurred?
What Windows component generated the event?
```

This makes ETW useful for **threat detection, incident response, and forensic investigation**.

---

## 🔍 Why Is ETW Useful for Security?

ETW can provide telemetry that is not always available in normal Windows logs.

It can collect information related to:

* Process creation and termination
* Network activity
* File activity
* Registry changes
* System activity
* Application activity
* Kernel activity

This additional information can help security teams:

* Detect suspicious behavior
* Investigate security incidents
* Understand what malware did on a system
* Perform forensic investigations
* Build better security detections

### 💡 Simple Example

Suppose a ransomware sample starts running.

A normal security log might show:

```text
ransomware.exe started
```

ETW can provide additional telemetry from different providers that may help investigators understand activities related to the process, files, registry, or network.

Therefore:

> 📌 **Important:**
> ETW is not simply another Windows event log. It is a **tracing system** that can collect events from different parts of Windows.

---

## 🧩 ETW Event Providers

One of the most important concepts in ETW is the **Event Provider**.

An Event Provider is a component that **generates events**.

Different Windows components and applications can have their own providers.

For example:

```text
Microsoft-Windows-Kernel-Process
Microsoft-Windows-Kernel-File
Microsoft-Windows-Kernel-Network
Microsoft-Windows-PowerShell
Microsoft-Windows-Kernel-Registry
```

Each provider focuses on different types of activity.

### 💡 Simple Example

Think of an ETW provider like a **security camera**.

Different cameras watch different places:

```text
Camera 1 → Process activity
Camera 2 → File activity
Camera 3 → Network activity
Camera 4 → Registry activity
```

In the same way, different ETW providers give us visibility into different parts of Windows.

---

# 🏗️ ETW Architecture & Components

The following diagram shows the main components of the ETW architecture.

<img width="477" height="480" alt="image" src="https://github.com/user-attachments/assets/fa08f09e-ea8a-4cc6-b5f1-55951ae67048" />


ETW uses several important components:

```text
                ┌──────────────┐
                │  Controller  │
                └──────┬───────┘
                       │
                 controls
                       │
                       ▼
                ┌──────────────┐
                │   Providers  │
                └──────┬───────┘
                       │
                    events
                       │
                       ▼
                ┌──────────────┐
                │   Channels   │
                └──────┬───────┘
                       │
                       ▼
                ┌──────────────┐
                │   Consumers  │
                └──────┬───────┘
                       │
                       ▼
                  ETL Files
```

---

## 🎛️ 1. Controllers

A **Controller** manages ETW tracing sessions.

It can:

* Start a tracing session
* Stop a tracing session
* Enable providers
* Disable providers
* Control which providers are collecting data

One commonly used controller is:

```text
logman.exe
```

### 💡 Simple Example

Think of the controller as the person who controls a camera system.

The controller decides:

```text
Start recording
Stop recording
Turn Camera A on
Turn Camera B off
```

In ETW:

```text
Controller → controls ETW sessions and providers
```

---

## 📡 2. Providers

**Providers generate events.**

Applications, Windows components, and drivers can register ETW providers.

There are four main types of ETW providers:

### `MOF Providers`

MOF stands for **Managed Object Format**.

These providers use predefined MOF schemas to describe events.

### `WPP Providers`

WPP stands for **Windows Software Trace Preprocessor**.

These providers use special macros and annotations in source code to generate tracing events.

They are often used for low-level tracing and debugging.

### `Manifest-based Providers`

These providers use XML manifest files to define event information.

This makes the event structure easier to manage.

### `TraceLogging Providers`

TraceLogging provides a simpler way for applications to generate ETW events.

It uses the TraceLogging API and requires less code to create events.

---

## 👀 3. Consumers

A **Consumer receives ETW events** and processes or analyzes them.

For example, a consumer can:

* Read ETW events
* Process the events
* Analyze the events
* Store the events

Events can also be written to an:

```text
.ETL
```

file.

### 💡 Simple Example

Think about a security camera again:

```text
Provider  → Camera
Controller → Person controlling the camera
Consumer  → Person watching the recording
```

The provider produces the information, while the consumer receives and analyzes it.

---

## 📦 4. Channels

**Channels** help organize and filter events.

They act as logical containers for events.

Different channels can contain different types of events based on their purpose and importance.

Consumers can subscribe to the channels that contain the events they are interested in.

### Example

A provider may have channels such as:

```text
Operational
Diagnostic
```

The channels can help separate different types of event information.

---

## 💾 5. ETL Files

ETW can save events to **Event Trace Log files**, commonly called:

```text
ETL files
```

These files provide persistent storage for ETW events.

They can be useful for:

* Offline analysis
* Long-term storage
* Incident investigation
* Forensic analysis

---

# ⚠️ Important ETW Notes

> 📌 **Important:**

* ETW supports providers running in both **kernel mode** and **user mode**.
* Some providers generate a very large number of events.
* Because of the amount of data they generate, some providers are disabled by default.
* Providers can be enabled when a tracing session needs them.
* ETW can be extended by creating **custom event providers**.
* Only ETW provider events that have a **Channel** property can be consumed by the Windows Event Log.

---

# 🛠️ Interacting With ETW

One of the most useful built-in tools for working with ETW is:

```text
logman.exe
```

`logman.exe` can be used to:

* Create tracing sessions
* Start tracing sessions
* Stop tracing sessions
* Investigate tracing sessions
* View providers
* View information about providers

---

## 🔎 Viewing ETW Sessions

The `-ets` parameter is important when working directly with Event Tracing Sessions.

For example:

```cmd
logman.exe query "EventLog-System" -ets
```

<img width="815" height="781" alt="image" src="https://github.com/user-attachments/assets/54b86206-fb07-477d-ba5c-522e0dd8910f" />


The `-ets` parameter tells `logman` to query the **Event Tracing Sessions**.

Without `-ets`, `logman` will not identify the ETW tracing session in the same way.

### 💡 Simple Example

You can think of:

```text
logman.exe query
```

as asking:

> "What information can you show me?"

While:

```text
logman.exe query ... -ets
```

is specifically asking about:

> "What Event Tracing Sessions are currently available?"

---

## 🔍 Information From an ETW Session

When we examine an ETW session, we can find information such as:

* `Name`
* `Max Log Size`
* `Log Location`
* Subscribed providers

This information can be useful during incident response.

For example, if an existing ETW session is collecting information from a provider related to the activity we are investigating, that session may contain useful evidence.

---

## 🆔 Provider Information

For each provider subscribed to a session, we can obtain information such as:

### `Name / Provider GUID`

The provider name and its unique GUID identify the provider.

### `Level`

The level controls the type or importance of events being collected.

For example:

```text
Warning
Informational
Critical
```

### `Keywords`

Keywords can be used to filter events based on the type of activity generated by the provider.

---

# 🔎 Finding ETW Providers

We can use `logman` to list providers available on the system.

```cmd
logman.exe query providers
```

<img width="767" height="818" alt="image" src="https://github.com/user-attachments/assets/e9dda10e-2f8c-4f0e-886d-356c60f46ce8" />


Windows 10 includes **more than 1,000 built-in providers**.

Third-party software can also create its own ETW providers.

Because there can be many providers, searching through the complete list can be difficult.

We can use `findstr` to filter the results.

### Example

```cmd
logman.exe query providers | findstr Winlogon
```

This can help us find providers related to `Winlogon`.

<img width="683" height="101" alt="image" src="https://github.com/user-attachments/assets/36650d15-d215-400b-8a47-f3bdf9df2fea" />


---

# 🧪 Investigating a Specific Provider

After finding a provider, we can use `logman` to learn more about it.

For example:

```cmd
logman.exe query providers Microsoft-Windows-Winlogon
```

<img width="649" height="719" alt="image" src="https://github.com/user-attachments/assets/61dc93c9-cb10-4377-83db-4a4a8d63e0d7" />


This can show information about:

* Provider keywords
* Event levels
* Processes currently using the provider

For example, the following keywords are associated with the provider:

```text
Microsoft-Windows-Winlogon/Diagnostic
Microsoft-Windows-Winlogon/Operational
```

These keywords reference event logs generated from this provider.

---

# 🖥️ GUI Tools for ETW

ETW does not have to be managed only from the command line.

There are also graphical tools.

## 1. Performance Monitor

The Windows **Performance Monitor** tool can be used to view running trace sessions.

You can select a trace session and view information such as:

* Providers
* Enabled features
* Trace information

You can also modify existing sessions by adding or removing providers.

New sessions can also be created under:

```text
User Defined
```

<img width="1473" height="1745" alt="image" src="https://github.com/user-attachments/assets/03ce6bc3-4d47-44ff-8a52-c7bc2e14adb7" />

<img width="1599" height="1121" alt="image" src="https://github.com/user-attachments/assets/e0610272-fd41-4764-93cb-ca08a01ce9f2" />


---

## 2. EtwExplorer

**EtwExplorer** is another tool that can be used to view ETW provider metadata.

<img width="1000" height="585" alt="image" src="https://github.com/user-attachments/assets/b6e913d2-7efa-4ae0-bca1-a3bf1b3085e9" />


---

# 🛡️ Useful ETW Providers

Different providers can help investigate different types of security activity.

| ETW Provider                                             | Useful For                                                      |
| -------------------------------------------------------- | --------------------------------------------------------------- |
| `Microsoft-Windows-Kernel-Process`                       | Process-related activity, including suspicious process behavior |
| `Microsoft-Windows-Kernel-File`                          | File operations and suspicious file activity                    |
| `Microsoft-Windows-Kernel-Network`                       | Network activity and suspicious connections                     |
| `Microsoft-Windows-SMBClient/SMBServer`                  | SMB and file-sharing activity                                   |
| `Microsoft-Windows-DotNETRuntime`                        | .NET application activity                                       |
| `OpenSSH`                                                | SSH connections and authentication activity                     |
| `Microsoft-Windows-VPN-Client`                           | VPN connection activity                                         |
| `Microsoft-Windows-PowerShell`                           | PowerShell execution and command activity                       |
| `Microsoft-Windows-Kernel-Registry`                      | Registry operations and changes                                 |
| `Microsoft-Windows-CodeIntegrity`                        | Code and driver integrity activity                              |
| `Microsoft-Antimalware-Service`                          | Antimalware service activity                                    |
| `WinRM`                                                  | Windows Remote Management activity                              |
| `Microsoft-Windows-TerminalServices-LocalSessionManager` | Remote Desktop activity                                         |
| `Microsoft-Windows-Security-Mitigations`                 | Security mitigation activity                                    |
| `Microsoft-Windows-DNS-Client`                           | DNS activity                                                    |
| `Microsoft-Antimalware-Protection`                       | Antimalware protection activity                                 |

### 💡 Simple Security Examples

#### Process Investigation

```text
Microsoft-Windows-Kernel-Process
```

can provide process-related information.

This can be useful when investigating suspicious process behavior.

#### PowerShell Investigation

```text
Microsoft-Windows-PowerShell
```

can provide information related to PowerShell activity.

This can help when investigating suspicious PowerShell execution.

#### Registry Investigation

```text
Microsoft-Windows-Kernel-Registry
```

can provide visibility into registry operations.

This can be useful when investigating suspicious registry changes.

#### Network Investigation

```text
Microsoft-Windows-Kernel-Network
```

can provide network-related telemetry.

This can help investigate suspicious network connections.

---

# 🔐 Restricted Providers

Some ETW providers are **restricted**.

This means that they contain sensitive information and cannot simply be accessed by every process.

Access requires the appropriate permissions.

One important restricted provider is:

```text
Microsoft-Windows-Threat-Intelligence
```

This provider can provide valuable information for:

* Threat detection
* Digital Forensics and Incident Response (DFIR)
* Investigation of sophisticated attacks

---

## 🛡️ Protected Process Light (PPL)

Access to the `Microsoft-Windows-Threat-Intelligence` provider requires a privileged process using **Protected Process Light (PPL)**.

PPL is a Windows security mechanism that protects certain important processes.

The original article explains that becoming a PPL anti-malware process requires an anti-malware vendor to go through Microsoft's approval and signing process.

This means accessing some restricted ETW information is **not simple**.

> 📌 **Important:**
> Restricted providers exist because the information they provide can be highly sensitive.

---

# 🚨 Why Is Microsoft-Windows-Threat-Intelligence Important?

The `Microsoft-Windows-Threat-Intelligence` provider can provide detailed information about potential threats.

This can be valuable during DFIR investigations because its telemetry may help investigators understand:

* Where a threat came from
* Which systems it interacted with
* Which data it interacted with
* What changes it made
* What happened during an attack

Real-time monitoring of this telemetry can also help security teams identify threats while they are occurring.

---

# 🧠 ETW and Sysmon

One important idea from this article is that **ETW can provide additional visibility when Sysmon alone does not capture everything an investigator needs**.

For example:

```text
                    Windows System
                         │
          ┌──────────────┴──────────────┐
          │                             │
        Sysmon                         ETW
          │                             │
     Security logs               ETW Providers
          │                             │
          └──────────────┬──────────────┘
                         │
                         ▼
                    Investigation
```

### 💡 Simple Example

Imagine that an attacker performs an action that Sysmon does not record in enough detail.

Instead of stopping the investigation, a defender can ask:

> "Is there an ETW provider that can give me more information about this activity?"

For example:

```text
Suspicious PowerShell
        ↓
Sysmon logs
        ↓
Need more information?
        ↓
Microsoft-Windows-PowerShell
        ↓
Additional ETW telemetry
```

This is why understanding ETW is useful for a **blue team / SOC analyst**.

---

# 📌 Key Takeaways

1. **ETW is a Windows tracing system** that collects events from applications, the kernel, and other components.

2. **ETW providers generate events.** Different providers provide visibility into different activities.

3. **Controllers manage ETW sessions.** `logman.exe` is one example of a controller.

4. **Consumers receive and process ETW events.**

5. **ETL files** can store ETW events for later investigation.

6. `logman.exe` can help analysts investigate ETW sessions and providers.

7. Windows contains many built-in ETW providers, and third-party applications can also create providers.

8. Different providers can help investigate **processes, files, networks, PowerShell, registry activity, DNS, VPN, SMB, and other activities**.

9. Some providers are **restricted** because they contain sensitive security telemetry.

10. ETW can provide **additional visibility for threat detection and incident response**, especially when normal logs or Sysmon do not provide enough information.

---

## 📚 References

* [A Primer on Event Tracing for Windows (ETW)](https://nasbench.medium.com/a-primer-on-event-tracing-for-windows-etw-997725c082bf)
* [A Beginner's All-Inclusive Guide to ETW](https://bmcder.com/blog/a-begginers-all-inclusive-guide-to-etw)
