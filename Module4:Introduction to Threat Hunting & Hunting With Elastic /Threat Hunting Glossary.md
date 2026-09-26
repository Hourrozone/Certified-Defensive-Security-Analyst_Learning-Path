# 🛡️ Threat Hunting Glossary

This glossary explains important terms used in **cybersecurity, Cyber Threat Intelligence (CTI), and threat hunting**.

---

## 🧑‍💻 1. Adversary

An **adversary** is a person, group, or organization that tries to gain unauthorized access to your organization.

They usually have a goal, such as:

* 💰 Making money
* 🕵️ Stealing insider information
* 📄 Stealing intellectual property
* 🎯 Attacking a specific organization

Adversaries have different levels of technical skill and resources. They try to bypass security controls to achieve their objectives.

### Common types of adversaries

* **Cyber criminals** — Attack for financial or other criminal benefits.
* **Insider threats** — People inside an organization who misuse their access.
* **Hacktivists** — Attack for political or social causes.
* **State-sponsored operators** — Groups supported or associated with governments.

📌 **Key idea:**
An adversary is simply the **person or group behind the attack**.

---

# 🚨 2. Advanced Persistent Threat (APT)

An **Advanced Persistent Threat (APT)** usually refers to a highly organized group or nation-state that has significant resources.

They may:

* Have a lot of funding
* Have many people working together
* Spend a long time attacking a target
* Focus on high-value organizations

Common targets can include:

* Government organizations
* Healthcare infrastructure
* Defense systems

### What does "Advanced Persistent Threat" really mean?

The word **Advanced** does not always mean the attacker uses the newest or most advanced technology.

It can refer to their:

* Strategic planning
* Organization
* Resources
* Ability to use different techniques

**Persistent** means they continue trying to achieve their objectives over a long period.

They may have enough:

* 💰 Money
* 👥 People
* ⏳ Time
* 🛠️ Resources

to continue their operations.

📌 **Easy way to remember:**

> **APT = Organized + Resourced + Persistent**

---

# 🎯 3. Tactics, Techniques, and Procedures (TTPs)

**TTPs** describe the way an adversary conducts operations.

They are often described as the adversary's **operational pattern or signature**.

TTP stands for:

* **Tactics**
* **Techniques**
* **Procedures**

---

## 🎯 Tactics

**Tactics** describe the adversary's **high-level objective**.

It answers:

> **"Why are they doing this?"**

For example:

* Gain initial access
* Maintain persistence
* Steal credentials
* Move laterally
* Exfiltrate data

---

## 🛠️ Techniques

**Techniques** describe the **general method** used to achieve a tactical objective.

It answers:

> **"How are they doing it?"**

A technique does not normally provide every individual step. Instead, it describes the general approach.

---

## 📋 Procedures

**Procedures** are the detailed steps used to perform a technique.

It is basically the:

> **"Recipe"**

for carrying out the technique.

### Simple example

| Level         | Example                                                                  |
| ------------- | ------------------------------------------------------------------------ |
| **Tactic**    | Gain Initial Access                                                      |
| **Technique** | Phishing                                                                 |
| **Procedure** | Send a specific malicious email with a link and use a particular payload |

📌 **Easy way to remember:**

> **Tactic = Why**
> **Technique = How**
> **Procedure = Exact steps**

---

## 🔍 Why TTPs Matter

Analyzing TTPs can help us understand:

* How an adversary enters a network
* How they move through the network
* How they maintain access
* How they achieve their objectives

Understanding TTPs can also help security teams create **Indicators of Compromise (IOCs)** that can be used to detect similar attacks.

---

# 🔎 4. Indicator

An **indicator** is information that can help security teams understand or identify suspicious activity.

In Cyber Threat Intelligence, an indicator can contain:

* Technical information
* Contextual information

Technical data by itself may not be very useful.

For example:

> `185.XXX.XXX.10`

is just an IP address.

But if we know that the IP address was:

* Used by malware
* Connected to a known attack
* Associated with a specific adversary
* Seen during a suspicious event

then the information becomes much more useful.

📌 **Key idea:**

> **Indicator + Context = More useful intelligence**

<img width="866" height="91" alt="image" src="https://github.com/user-attachments/assets/45c193b2-3c0e-4786-8d47-093ceb0971eb" />

---

# ⚠️ 5. Threat

A **threat** can be understood using three important factors:

1. **Intent**
2. **Capability**
3. **Opportunity**

<img width="446" height="451" alt="image" src="https://github.com/user-attachments/assets/97f3e116-3902-4354-939a-37142367fc86" />

---

## 🎯 Intent

**Intent** describes **why** an adversary wants to attack.

Examples include:

* Corporate espionage
* Financial gain
* Cybercrime
* Stealing information
* Targeting business relationships

---

## 🛠️ Capability

**Capability** describes what the adversary **has the ability and resources to do**.

This can include:

* Tools
* Malware
* Technical skills
* Money
* People
* Infrastructure

The adversary's technical skills and available resources affect how capable they are of carrying out an attack.

---

## 🚪 Opportunity

**Opportunity** refers to conditions that make an attack possible or easier.

Examples include:

* Obtaining employee email addresses
* Obtaining stolen credentials
* Finding vulnerable software
* Discovering exposed systems
* Learning about weaknesses in an organization

📌 **Easy way to remember:**

> **Intent = Why**
> **Capability = Can they do it?**
> **Opportunity = Do they have a chance to do it?**

---

# 🗂️ 6. Campaign

A **campaign** is a collection of related incidents that appear to be connected.

These incidents may have:

* Similar TTPs
* Similar objectives
* Similar collection requirements

For example, several organizations may report attacks that use similar malware, infrastructure, and techniques.

When analysts connect these incidents together, they may identify them as part of the same campaign.

📌 Campaign intelligence can take significant **time and effort** to collect and analyze because information may come from many different incidents and organizations.

---

# 🚨 7. Indicators of Compromise (IOCs)

**Indicators of Compromise (IOCs)** are digital traces or artifacts associated with malicious activity.

They can act like **signposts** that help security teams identify an attack.

### Examples of IOCs

* File hashes
* Suspicious IP addresses
* Malicious URLs
* Malicious domain names
* Malicious executable names
* Malicious script names

### Example

A malware sample might have this SHA-256 hash:

```text
<malicious-file-hash>
```

A security team can search for this hash across endpoints to see whether the malware exists elsewhere.

📌 **Why IOCs are useful**

Continually collecting, tracking, and analyzing IOCs can help organizations:

* Detect threats faster
* Investigate incidents
* Identify similar attacks
* Respond more effectively

---

# 🔺 8. Pyramid of Pain

The **Pyramid of Pain** is a visualization that shows different types of indicators used to detect adversaries.

It also shows:

* How difficult each type of indicator is to obtain
* How difficult it is for an adversary to change that indicator
* How much pain or effort the attacker experiences when defenders detect higher-level indicators

The concept was introduced by **David Bianco** from FireEye in his presentation:

> *Intel-Driven Detection and Response to Increase Your Adversary's Cost of Operations*

As we move **higher** in the Pyramid of Pain:

* Indicators become harder for defenders to obtain and use.
* They become harder for attackers to change.
* They provide more useful information about the adversary.

<img width="769" height="447" alt="image" src="https://github.com/user-attachments/assets/88601cdd-8027-45d6-a958-3713994b5c4e" />

---

# #️⃣ 9. Hash Values

A **hash value** is like a **digital fingerprint of a file**.

Hash algorithms such as:

* MD5
* SHA-1
* SHA-256

take an input and produce a fixed-size value.

For example:

```text
File → Hash Algorithm → Hash Value
```

A malware file can therefore be identified using its hash.

### ⚠️ Limitation of hashes

A very small change to a file can produce a completely different hash.

For example:

```text
Original malware → Hash A
```

After changing one small part:

```text
Modified malware → Hash B
```

Therefore, hashes are easy for attackers to change and are considered a **lower-level indicator** in the Pyramid of Pain.

📌 **Key idea:**

> A hash can identify a specific file, but changing the file can change the hash.

---

# 🌐 10. IP Addresses

An **IP address** identifies a device or network location.

Security teams can use IP addresses to investigate:

* Network connections
* Suspicious traffic
* Possible attack sources
* Command-and-control connections

### ⚠️ Limitation

Attackers can hide or change their apparent IP address using methods such as:

* IP spoofing
* VPNs
* Proxies
* TOR

Therefore, IP addresses can be relatively easy for attackers to change or hide.

---

# 🌍 11. Domain Names

A **domain name** is used to identify one or more IP addresses.

For example:

```text
www.example.com
```

can point to one or more IP addresses.

### ⚠️ How attackers avoid detection

Attackers may use:

### Domain Generation Algorithms (DGAs)

A **DGA** can generate many pseudo-random domain names.

For example:

```text
abc123example.com
x7k29example.net
q91example.org
```

This can make it harder for defenders to block every malicious domain.

### Dynamic DNS

Attackers can also use dynamic DNS services to quickly change the IP address associated with a domain.

📌 **Key idea:**

> Domain names can also be changed, so they are not as difficult for attackers to replace as higher-level indicators.

---

# 🖥️ 12. Network and Host Artifacts

Artifacts are traces left behind by activity.

There are two important types:

1. **Network Artifacts**
2. **Host Artifacts**

---

## 🌐 Network Artifacts

Network artifacts are traces of attacker activity found within network infrastructure.

They can appear in:

* Network logs
* Packet captures
* NetFlow data
* DNS logs

Examples include:

* Unusual network traffic patterns
* Unique packet headers
* Unusual protocol usage

These artifacts can be more difficult for attackers to change without affecting their operation or making their activity less effective.

---

## 💻 Host Artifacts

Host artifacts are traces left on an individual computer or endpoint.

They can be found in:

* System logs
* File systems
* Windows Registry
* Running processes
* Loaded DLLs
* Volatile memory

Examples:

```text
Unusual Registry entry
Suspicious file path
Unknown running process
Unexpected DLL
```

These artifacts can also be difficult for attackers to completely change without affecting their attack or exposing themselves.

---

## 🔍 Why Artifacts Matter

Analyzing network and host artifacts can help defenders understand:

* What tools the attacker used
* What techniques they used
* What procedures they followed
* How the attack happened

📌 In the Pyramid of Pain, **Network Artifacts** and **Host Artifacts** are higher than hashes, IP addresses, and domains.

This means they can be more difficult for attackers to change or hide without affecting their operation.

---

# 🛠️ 13. Tools

**Tools** are software used by adversaries to perform attacks.

Examples include:

* Malware
* Exploits
* Scripts
* Command-and-Control (C2) frameworks

Identifying an attacker's tools can help defenders understand:

* Their capabilities
* Their methods
* Their possible objectives

### ⚠️ Limitation

Sophisticated adversaries may:

* Create their own tools
* Modify existing tools
* Customize tools
* Change tools to avoid detection

Therefore, detecting a specific tool is not always enough.

---

# 🎯 14. TTPs in the Pyramid of Pain

**TTPs are at the top of the Pyramid of Pain.**

They represent how an adversary actually performs their operations.

### Tactics

Describe the overall objective.

### Techniques

Describe the actions or methods used to achieve the objective.

### Procedures

Describe the exact steps used to perform the technique.

For example:

| Level         | Example                                                            |
| ------------- | ------------------------------------------------------------------ |
| **Tactic**    | Initial Access                                                     |
| **Technique** | Spear-phishing                                                     |
| **Procedure** | Specific steps used to send and deliver a malicious phishing email |

Another example could involve exploiting a software vulnerability:

```text
Tactic
↓
Gain Initial Access

Technique
↓
Exploit a Vulnerability

Procedure
↓
Specific steps used to exploit that vulnerability
```

📌 **Why TTPs are important**

TTPs provide valuable information about **how an adversary operates**.

They are also difficult for an adversary to completely change because changing their entire way of operating can require significant:

* Time
* Resources
* Training
* Effort

---

# 💎 15. Diamond Model

The **Diamond Model of Intrusion Analysis** is a framework used to understand and analyze cyber intrusions.

It was developed by:

* Sergio Caltagirone
* Andrew Pendergast
* Christopher Betz

The model has **four main components**, represented as the four points of a diamond:

1. Adversary
2. Capability
3. Infrastructure
4. Victim

<img width="749" height="542" alt="image" src="https://github.com/user-attachments/assets/4d51de2f-e265-4080-82be-9dbf758f9367" />

---

## 🧑‍💻 Adversary

The **Adversary** is the person, group, or organization responsible for the attack.

Security teams may want to understand:

* Who they are
* Their capabilities
* Their motivation
* Their intent

---

## 🛠️ Capability

**Capability** represents the tools and methods the adversary uses.

It can include:

* Malware
* Exploits
* Scripts
* TTPs
* Other malicious tools

It also includes the methods used to deploy those tools.

---

## 🌐 Infrastructure

**Infrastructure** represents the resources used by the adversary to support the attack.

Examples include:

* Servers
* Domain names
* IP addresses
* Other network resources

Infrastructure can be used to:

* Deliver malware
* Control compromised systems
* Communicate with infected systems
* Exfiltrate stolen data

---

## 🎯 Victim

The **Victim** is the target of the attack.

It could be:

* A person
* An organization
* A computer
* A network
* A specific system

Understanding the victim helps defenders understand:

* Vulnerabilities
* Valuable assets
* Exposure to threats

---

# 🔗 16. Relationships in the Diamond Model

The four components are connected by **bidirectional relationships**.

A simple way to understand the relationship is:

```text
                 Adversary
                     ↕
                 Capability
                     ↕
               Infrastructure
                     ↕
                   Victim
```

In a real intrusion, these components interact with each other.

For example:

> An adversary uses a capability through infrastructure to attack a victim.

The model helps analysts understand the relationships between the different parts of an attack.

---

# 🏦 17. Diamond Model Example

Imagine a **financial institution** is attacked.

### Step 1 — Victim

```text
Financial Institution
```

The financial institution is the **Victim**.

### Step 2 — Adversary

```text
Cybercriminal Group
```

The cybercriminal group is the **Adversary**.

### Step 3 — Capability

The group uses:

```text
Spear-phishing emails
Banking Trojan
```

These represent the **Capability**.

### Step 4 — Infrastructure

The emails are sent using a:

```text
Botnet
```

The botnet represents part of the attacker's **Infrastructure**.

### Attack flow

```text
Cybercriminal Group
        ↓
Spear-phishing Email
        ↓
Botnet Infrastructure
        ↓
Financial Institution
        ↓
Victim clicks malicious link
        ↓
Banking Trojan installed
        ↓
Sensitive financial data stolen
```

---

# 🔍 18. Why Use the Diamond Model?

The Diamond Model helps defenders understand the relationship between:

```text
Adversary
    ↕
Capability
    ↕
Infrastructure
    ↕
Victim
```

By analyzing these relationships, security teams can better understand the threat.

For example, a financial institution could:

* Improve email security
* Monitor for the specific banking Trojan
* Monitor suspicious infrastructure
* Detect unusual network activity
* Investigate related indicators

📌 **Key idea:**

> If you understand one part of the diamond, you may be able to discover information about the other parts.

---

# ⚔️ 19. Diamond Model vs. Cyber Kill Chain

The **Diamond Model** and **Cyber Kill Chain** provide different views of a cyber attack.

### Cyber Kill Chain

The Cyber Kill Chain mainly focuses on the **stages of an attack**.

For example:

```text
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

### Diamond Model

The Diamond Model focuses more on the **components involved in an intrusion**:

```text
        Adversary
           ↕
       Capability
           ↕
     Infrastructure
           ↕
         Victim
```

📌 **Simple difference:**

> **Cyber Kill Chain = What stage of the attack?**

> **Diamond Model = Who, what, where, and who is being targeted?**

Both models can provide useful perspectives for cybersecurity analysis.

---

# 🧠 20. Quick Glossary

| Term                 | Simple Meaning                                                         |
| -------------------- | ---------------------------------------------------------------------- |
| **Adversary**        | Person or group carrying out an attack                                 |
| **APT**              | Highly organized and persistent threat group                           |
| **Tactic**           | The adversary's objective — **Why?**                                   |
| **Technique**        | General method — **How?**                                              |
| **Procedure**        | Detailed steps used to perform a technique                             |
| **Indicator**        | Information that helps identify suspicious activity                    |
| **Threat**           | Intent + Capability + Opportunity                                      |
| **Campaign**         | Collection of related incidents                                        |
| **IOC**              | Digital trace associated with malicious activity                       |
| **Hash**             | Digital fingerprint of a file                                          |
| **IP Address**       | Network identifier that can be used as an indicator                    |
| **Domain**           | Name used to identify network resources                                |
| **Network Artifact** | Evidence left in network activity                                      |
| **Host Artifact**    | Evidence left on a computer or endpoint                                |
| **Tool**             | Software used by an attacker                                           |
| **TTPs**             | How an adversary operates                                              |
| **Pyramid of Pain**  | Shows how difficult different indicators are to change                 |
| **Diamond Model**    | Framework connecting Adversary, Capability, Infrastructure, and Victim |

---

# ✅ Key Takeaways

1. **Adversary** = the person or group behind the attack.
2. **APT** = organized and persistent threat activity supported by significant resources.
3. **TTPs** describe how an adversary operates.
4. **Tactic = Why**, **Technique = How**, **Procedure = Exact steps**.
5. **IOC** = a digital trace that can help identify malicious activity.
6. **Threat** can be understood through **Intent + Capability + Opportunity**.
7. **Pyramid of Pain** shows different levels of indicators and how difficult they are for attackers to change.
8. **Hashes, IPs, and domains** are useful but can often be changed.
9. **Network artifacts, host artifacts, and TTPs** can provide deeper insight into attacker behavior.
10. **Diamond Model** helps analysts understand the relationship between **Adversary, Capability, Infrastructure, and Victim**.
11. **Cyber Kill Chain** focuses on attack stages, while the **Diamond Model** focuses on the components and relationships involved in an intrusion.

> 💡 **For threat hunting:**
> Don't only ask **"What IOC should I search for?"**
> Also ask **"What is the adversary trying to do, how do they normally do it, and what traces would that activity leave in my environment?"**
