# 🛡️ Threat Hunting Fundamentals

---

# 1. Threat Hunting Definition

## 🧠 What is Threat Hunting?

A security breach can sometimes stay hidden inside a network for **weeks or even months** before the organization discovers it.

The time between:

> **Attacker gets into the system → Organization discovers the attack**

is called **Dwell Time**.

### 📌 Dwell Time

**Dwell time** = How long an attacker stays inside a system before being detected.

For example:

```text
Attacker enters network
        ↓
      3 weeks
        ↓
Security team discovers the attacker
```

The longer the attacker stays undetected, the more damage they may be able to cause.

---

## 🛡️ Why Do We Need Threat Hunting?

Traditional security is often **reactive**.

That means:

```text
Security tool detects something
        ↓
Security team receives alert
        ↓
Analyst investigates
```

But what if the attacker does something that the security tools do not detect?

This is where **Threat Hunting** becomes important.

Threat hunting is a **proactive** activity where security professionals actively search for threats that may already be inside the environment but have not been detected.

---

## 🔍 Simple Definition

**Threat hunting** is:

> An active, human-led process of searching through security data to find hidden or suspicious threats that normal security tools may have missed.

Threat hunters do not simply wait for alerts.

They actively ask:

> **"Could an attacker already be inside our network?"**

Then they search for evidence.

---

# 2. Main Goal of Threat Hunting

The main goal is to **reduce dwell time**.

Instead of allowing an attacker to stay inside the network for weeks, threat hunting tries to discover the attacker as early as possible.

For example:

```text
Without Threat Hunting:

Attacker enters
      ↓
      ↓
      ↓
      ↓
Several weeks
      ↓
Detection


With Threat Hunting:

Attacker enters
      ↓
Threat Hunt
      ↓
Suspicious activity discovered
      ↓
Investigation
      ↓
Response
```

### 💡 Key Idea

The earlier we discover an attacker, the less time they have to:

* Move through the network
* Steal information
* Install malware
* Gain higher privileges
* Establish persistence
* Cause damage

---

# 3. How Does Threat Hunting Work?

A threat hunting process can start with identifying important systems and data.

### Step 1 — Identify Important Assets

First, identify systems or data that attackers may want to target.

Examples:

* Important servers
* Databases
* User accounts
* Financial information
* Critical applications
* Network infrastructure

These are sometimes called **high-value assets** or **crown jewels**.

---

### Step 2 — Understand Attacker TTPs

Next, we think about how attackers may attack these systems.

This includes **TTPs**:

> **Tactics, Techniques, and Procedures**

Simply:

* **Tactics** → What the attacker wants to achieve
* **Techniques** → How the attacker achieves it
* **Procedures** → The specific way they perform the technique

For example:

```text
Goal:
Steal credentials

Technique:
Credential Dumping

Procedure:
Use a tool to dump credentials from LSASS
```

---

### Step 3 — Search for Evidence

The threat hunter searches security data for evidence related to those attacker behaviors.

Examples:

* Suspicious processes
* Strange PowerShell commands
* Unusual network connections
* New files
* Suspicious registry changes
* Unexpected user activity
* Known malicious IP addresses
* Known malicious file hashes

These pieces of evidence can be called **artifacts** or **Indicators of Compromise (IoCs)**.

---

### Step 4 — Compare With Normal Behavior

Threat hunters also need to understand what is **normal** in the environment.

For example:

```text
Normal:
User normally logs in from Cambodia.

Suspicious:
The same account suddenly logs in from another unusual location.
```

The hunter investigates the unusual activity.

---

# 4. Threat Intelligence and Threat Hunting

**Threat Intelligence** is very useful for threat hunting.

Threat intelligence provides information about:

* Attackers
* Malware
* IP addresses
* Domains
* File hashes
* TTPs
* Vulnerabilities
* Previous attacks

This information can help threat hunters create a **hunting hypothesis**.

### 🧠 Example

Threat intelligence tells us:

> A known attacker is using PowerShell to download malware.

The threat hunter can create a hypothesis:

> **"The attacker may be using PowerShell to download files inside our network."**

Then the hunter searches the logs for suspicious PowerShell activity.

---

# 5. Important Parts of Threat Hunting

Threat hunting includes several important skills and activities.

### 🛡️ 1. Proactive Searching

Instead of waiting for alerts, hunters actively search for possible threats.

They use:

* Hypotheses
* Threat intelligence
* Attacker TTPs
* Security logs

---

### 🔍 2. Reactive Searching

Threat hunting can also happen after something suspicious has already been discovered.

For example:

```text
SIEM detects suspicious activity
        ↓
Incident confirmed
        ↓
Threat hunter searches the network
        ↓
Finds other systems with the same activity
```

The goal is to understand how far the attack has spread.

---

### 🧠 3. Understand the Threat Landscape

Threat hunters should understand:

* Common cyber threats
* Attackers
* Malware
* TTPs
* Cyber Kill Chain
* Common attack methods

---

### 🧠 4. Think Like an Attacker

A threat hunter should try to understand:

> **"If I were the attacker, what would I do next?"**

This helps the hunter find activity that may not be obvious.

---

### 🖥️ 5. Understand the Organization

A hunter needs to understand the organization's:

* Network
* Servers
* Applications
* Users
* Endpoints
* Important assets
* Normal activities

You cannot easily identify something abnormal if you do not understand what is normal.

---

### 🛠️ 6. Use Good Data and Tools

Threat hunters need reliable security data.

Common sources include:

* SIEM
* EDR
* Network logs
* Windows Event Logs
* Sysmon
* Threat Intelligence platforms
* Other security monitoring tools

---

# 6. Relationship Between Incident Handling and Threat Hunting

Threat hunting and **Incident Handling** are closely connected.

They can support each other during different stages of an incident.

---

## 📌 Preparation

Before an incident happens, the organization should decide:

* What threat hunters are allowed to do
* When they should take action
* How investigations should be performed
* Who is responsible for each task
* What procedures should be followed

Threat hunting can be included in existing incident handling policies instead of creating completely separate policies.

---

## 🔍 Detection & Analysis

This is where threat hunters can be very useful.

Suppose the SOC detects a suspicious activity.

The threat hunter can:

* Investigate the activity
* Check whether it is really malicious
* Search for additional IoCs
* Search for other affected systems
* Find evidence that the first investigation may have missed

Example:

```text
SOC Alert
   ↓
Suspicious PowerShell
   ↓
Threat Hunter investigates
   ↓
Searches other endpoints
   ↓
Finds the same PowerShell activity
```

This can help reveal the **full scope of the attack**.

---

## 🚨 Containment, Eradication & Recovery

During this stage, the threat hunter's role depends on the organization.

Some organizations may ask threat hunters to help with:

* Containment
* Eradication
* Recovery

Other organizations may keep these responsibilities with the Incident Response team.

Therefore, the exact responsibilities should be defined by the organization's:

* Security policies
* Procedures
* Roles and responsibilities

---

## 📋 Post-Incident Activity

After the incident is finished, threat hunters can help improve security.

For example, they may recommend:

* New detection rules
* Better monitoring
* Better security controls
* New hunting queries
* Improvements to security processes

The goal is to make it harder for the same type of attack to succeed again.

---

# 7. Threat Hunting Team Structure

A threat hunting team can contain people with different skills.

Each person contributes something different to the investigation.

---

## 🔍 Threat Hunter

The main person responsible for actively searching for threats.

They should understand:

* Cyber threats
* Attacker TTPs
* IoCs
* Detection techniques
* Threat hunting tools

---

## 🧠 Threat Intelligence Analyst

Collects and analyzes information about threats.

Sources may include:

* Open-source intelligence
* Threat feeds
* Industry reports
* Dark web intelligence

They help answer questions such as:

> Who are the current threats?

> What techniques are attackers using?

> Which organizations are being targeted?

This information helps threat hunters create better hunting hypotheses.

---

## 🚨 Incident Responder

When a threat is discovered, incident responders investigate and respond to it.

Their responsibilities may include:

* Investigation
* Containment
* Eradication
* Recovery

Their goal is to control the incident and return systems to normal operation.

---

## 🔬 Forensics Expert

Forensics experts investigate technical evidence from an incident.

They may work with:

* Disk evidence
* Memory
* Malware
* Logs
* Attack artifacts

They can also perform:

* Digital forensics
* Malware analysis
* Reverse engineering
* Detailed incident reporting

---

## 📊 Data Analyst / Data Scientist

Works with large amounts of security data.

They can use:

* Statistics
* Machine learning
* Data mining
* Data analysis

Their goal is to find:

* Patterns
* Relationships
* Trends
* Unusual behavior

---

## 🛠️ Security Engineer / Architect

Responsible for designing and maintaining security infrastructure.

They work with:

* Security tools
* Networks
* Applications
* Security controls

They can also help threat hunters implement tools and defenses.

---

## 🌐 Network Security Analyst

Focuses on network traffic and network behavior.

They understand what normal network traffic looks like and can investigate unusual activity.

Example:

```text
Normal:
Workstation → Internal Server

Suspicious:
Workstation → Unknown External IP
```

---

## 👨‍💼 SOC Manager

The SOC Manager oversees the security operations team.

They help with:

* Team coordination
* Communication
* Operations
* Management
* Overall security monitoring

---

# 8. When Should We Hunt?

Threat hunting should not only happen after an incident.

It should be a **continuous and proactive activity**.

However, there are situations where threat hunting becomes especially important.

---

## 🚨 1. New Information About an Attacker or Vulnerability

Suppose a new vulnerability is discovered in software that the organization uses.

Threat hunters may search the environment to determine:

> **"Has anyone already exploited this vulnerability?"**

For example:

```text
New vulnerability discovered
        ↓
Organization uses affected software
        ↓
Threat Hunt
        ↓
Search for exploitation activity
```

---

## 🚨 2. New IoCs for a Known Attacker

Threat intelligence may provide new IoCs related to a known attacker.

Examples:

* New IP addresses
* New domains
* New file hashes
* New malware
* New TTPs

The threat hunter can search the organization's environment for those indicators.

Example:

```text
New malicious IP discovered
        ↓
Search SIEM logs
        ↓
Did any internal system communicate with it?
        ↓
Yes → Investigate
```

---

## 🚨 3. Multiple Network Anomalies

One unusual event may not always mean an attack.

But many unusual events happening around the same time may require investigation.

Example:

```text
Unusual login
      +
Unusual network connection
      +
Unknown process
      +
New suspicious file
      ↓
Threat Hunt
```

The goal is to understand whether these events are connected.

---

## 🚨 4. During an Incident

Threat hunting can happen while Incident Response is already handling an incident.

For example:

```text
Malware discovered on Computer A
             ↓
Incident Response
     → Contain Computer A
             ↓
Threat Hunting
     → Search other computers
             ↓
Find Computer B and C
```

This helps determine whether the attacker has compromised other systems.

---

## 🔄 5. Regular Proactive Hunting

Threat hunting should also happen regularly.

Why?

Because some threats may bypass existing security controls.

Regular hunting can help discover threats that have remained hidden.

### 💡 Key Idea

> **Do not wait for an alert to start hunting.**

Regular threat hunting helps find suspicious activity before it becomes a major incident.

---

# 9. Risk Assessment and Threat Hunting

Threat hunting is also connected to **Risk Assessment**.

Risk assessment helps an organization understand:

* What assets are important
* What threats exist
* What vulnerabilities exist
* What could happen if a vulnerability is exploited
* What risks should receive more attention

This information can help threat hunters decide **where to hunt first**.

---

# 10. How Risk Assessment Helps Threat Hunting

## 🎯 1. Prioritize Hunting

Not every system is equally important.

For example:

```text
Employee test computer
        ↓
Lower priority

Bank database
        ↓
Higher priority
```

Threat hunters can focus more attention on important assets, sometimes called **"crown jewels."**

Examples:

* Sensitive databases
* Critical applications
* Important servers
* Key network infrastructure

---

## 🧠 2. Understand the Threat Landscape

Risk assessment helps identify possible threats.

This helps threat hunters understand:

> Who might attack us?

> How might they attack us?

> What techniques might they use?

This information can be used to create hunting hypotheses.

---

## ⚠️ 3. Identify Vulnerabilities

Risk assessment can identify weaknesses in systems.

Threat hunters can then search for evidence that those weaknesses have been exploited.

Example:

```text
Vulnerability:
Application allows privilege escalation

        ↓

Threat Hunt:
Search for unusual privilege changes
```

---

## 🧠 4. Use Threat Intelligence More Effectively

Risk assessment helps identify which attackers and attack methods are more relevant to the organization.

Threat intelligence can then be used to search for those threats.

---

## 🚨 5. Improve Incident Response

Risk assessment can help organizations prepare for possible incidents.

For example:

```text
Identify major risk
       ↓
Understand possible attack
       ↓
Prepare response plan
       ↓
Threat Hunting
       ↓
Incident Response
```

This can help the organization respond more effectively.

---

## 🛡️ 6. Improve Security Controls

Risk assessment can also show where security controls need improvement.

For example:

```text
Risk Assessment
       ↓
Find security weakness
       ↓
Improve security control
       ↓
Better detection/prevention
```

---

# 11. Tools Used With Risk Assessment and Threat Hunting

Organizations can use different tools to support these activities.

Examples include:

* Vulnerability scanners
* Penetration testing tools
* Threat Intelligence platforms
* SIEM
* Security monitoring tools

### 🛠️ SIEM Example

A **SIEM** collects and correlates security events from different sources.

For example:

```text
Windows Logs
      +
Sysmon
      +
Firewall Logs
      +
Endpoint Logs
      ↓
     SIEM
      ↓
Threat Hunter searches the data
```

This gives the threat hunter a larger view of what is happening in the environment.

---

# 12. Simple Threat Hunting Workflow

You can remember the whole concept with this workflow:

```text
1. Understand the environment
          ↓
2. Identify important assets
          ↓
3. Understand threats and TTPs
          ↓
4. Create a hunting hypothesis
          ↓
5. Collect security data
          ↓
6. Search for suspicious activity
          ↓
7. Investigate the findings
          ↓
8. Confirm or reject the hypothesis
          ↓
9. Find affected systems
          ↓
10. Support Incident Response
          ↓
11. Improve detections and defenses
```

---

# 🧠 Simple Example

Imagine you receive threat intelligence saying:

> A ransomware group is using PowerShell to download malware.

A threat hunter could create this hypothesis:

> **"The attacker may have used PowerShell to download a malicious file in our environment."**

Then:

```text
Threat Intelligence
        ↓
Create Hypothesis
        ↓
Search PowerShell Logs
        ↓
Find Suspicious Command
        ↓
Check Network Connection
        ↓
Check Downloaded File
        ↓
Check File Hash
        ↓
Search Other Computers
        ↓
Confirm / Reject Threat
        ↓
Support Incident Response
```

This is a simple example of **proactive threat hunting**.

---

# 📌 Important Terms to Remember

| Term                    | Simple Meaning                                              |
| ----------------------- | ----------------------------------------------------------- |
| **Threat Hunting**      | Actively searching for hidden threats                       |
| **Dwell Time**          | How long an attacker stays undetected                       |
| **TTPs**                | How attackers operate                                       |
| **IoC**                 | Evidence that may indicate compromise                       |
| **Threat Intelligence** | Information about threats and attackers                     |
| **Hunting Hypothesis**  | An idea about what suspicious activity we should search for |
| **Risk Assessment**     | Identifying and evaluating security risks                   |
| **Incident Response**   | Responding to and managing a security incident              |
| **SIEM**                | System that collects and analyzes security logs             |
| **Crown Jewels**        | The organization's most important assets                    |

---

# 💡 Key Takeaway

**Threat hunting = Don't wait for the alert. Go and look for the threat.**

A simple way to remember it:

> **Know your environment → Understand the attacker → Create a hypothesis → Search the data → Investigate → Find the threat → Help respond → Improve detection.**

The main purpose is to **find hidden threats earlier and reduce the time attackers can stay inside the environment.**
