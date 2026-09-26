# Threat Intelligence Fundamentals

## Cyber Threat Intelligence Definition

**Cyber Threat Intelligence (CTI)** is important information that helps an organization understand and defend against cyberattacks.

The main goal of a CTI team is to help the organization move from only **reacting to attacks** to being more **proactive** — meaning we try to understand and prepare for possible attacks before they happen.

CTI also provides useful information to the **Security Operations Center (SOC)** so security analysts can make better decisions.

There are **four fundamental principles** that make CTI useful:

<img width="502" height="229" alt="image" src="https://github.com/user-attachments/assets/0ad05459-4bc7-4c62-af21-bdf9873280fe" />

### 1. Relevance

There is a huge amount of cybersecurity information available from many sources, such as:

* Social media
* Security vendor reports
* Threat intelligence platforms
* Other organizations in the same industry
* Security researchers

However, not all information is useful to our organization.

**Relevance means that the intelligence should be related to our organization, systems, software, or business.**

For example, imagine a security report says that a vulnerability was discovered in a particular software.

If our organization and our trusted partners **do not use that software**, then the vulnerability may not be an urgent problem for us.

> 💡 **Key idea:** Information is valuable when it is relevant to our environment.

---

### 2. Timeliness

Threat intelligence needs to reach the security team **quickly**.

Fresh information is usually more useful because attackers can change their tools, infrastructure, and methods over time.

For example, an IP address that was used by an attacker six months ago might no longer be used today. The attacker may have changed their server or infrastructure.

This means that the value of some threat information **decreases as it gets older**.

> ⏰ **Key idea:** Fresh intelligence is usually more useful than old intelligence.

---

### 3. Actionability

CTI should give the security team information that they can **actually use to take action**.

For example, intelligence should help the team answer questions such as:

* What should we block?
* What should we investigate?
* What systems should we check?
* What security rule should we create?
* What users should we warn?

If intelligence does not tell us what action we can take, its value becomes lower.

The CTI analyst should continue analyzing information until it produces **relevant, timely, and actionable intelligence**.

> ⚠️ **Important:** Spending a lot of time analyzing information that never leads to useful action is not productive.

The article refers to this kind of unproductive cycle as a **"self-licking ice cream cone"** — a humorous term for a process that keeps itself going without producing a useful result.

---

### 4. Accuracy

Before sharing threat intelligence with the defense team, we need to make sure the information is **correct**.

Incorrect information can waste time and resources.

For example:

* An incorrect IP address could cause us to block a legitimate service.
* Incorrectly identifying an attacker could lead to the wrong conclusions.
* Incorrect Tactics, Techniques, and Procedures (TTPs) could cause analysts to search for the wrong behavior.

If we are not completely sure that information is accurate, we should include a **confidence level**.

This tells the security team how much they should trust the information.

> 🔎 **Key idea:** Intelligence should be checked before people use it to make security decisions.

---

### What Do We Gain From Good CTI?

When **Relevance, Timeliness, Actionability, and Accuracy** work together, CTI can help us:

* Understand possible attacker operations and campaigns targeting our organization.
* Improve our security data by analyzing information from CTI analysts and other defenders.
* Discover attacker **TTPs**, which helps us create better defenses.
* Understand how attackers behave.
* Give decision-makers useful information so they can make informed business and security decisions.

---

# The Difference Between Threat Intelligence & Threat Hunting

**Threat Intelligence** and **Threat Hunting** are two different areas of cybersecurity.

They have different purposes, but they work well together.

> ⚠️ **Important:** Threat Intelligence and Threat Hunting are **not replacements for each other**. They perform different jobs.

---

## Threat Intelligence (Predictive)

Threat Intelligence mainly tries to understand what an attacker **might do next**.

The goal is to understand:

* Where the attacker might attack.
* When the attack might happen.
* How the attacker might operate.
* What the attacker ultimately wants to achieve.

In simple terms, Threat Intelligence tries to answer:

> **"What might the attacker do, and how can we prepare for it?"**

---

## Threat Hunting (Reactive and Proactive)

Threat Hunting focuses on actively looking for signs that an attacker **is already inside the environment or was previously inside and avoided detection**.

A starting event might be:

* A security incident in our own network.
* A suspicious activity detected by security tools.
* An attack against another organization in the same industry.

The security team can then investigate the environment to determine whether the attacker is present or was previously present.

In simple terms, Threat Hunting asks:

> **"Is the attacker already here, or was the attacker here before?"**

---

## How Threat Intelligence and Threat Hunting Work Together

Threat Intelligence and Threat Hunting support each other.

For example:

**Threat Intelligence → Threat Hunting**

The CTI team studies attacker activity and creates information about the attacker.

That information can then be given to Threat Hunters.

Threat Hunters can use it to search the organization's environment for signs of the attacker.

**Threat Hunting → Threat Intelligence**

Threat Hunters may discover new information during their investigation.

That information can be sent back to CTI analysts.

The CTI team can then use it to improve their intelligence and make their understanding of the attacker more accurate.

> 🔄 **Key idea:** CTI helps Threat Hunters know what to look for, while Threat Hunting can provide new information that improves CTI.

---

# Criteria Of Cyber Threat Intelligence

What makes **Cyber Threat Intelligence (CTI)** valuable?

What problems can it help solve?

As discussed earlier, effective CTI should be:

* **Actionable**
* **Timely**
* **Relevant**
* **Accurate**

These four qualities are the foundation of useful CTI.

Good CTI helps an organization gain better visibility into **what attackers are doing**.

It can also provide additional benefits, such as:

* Better understanding of threats against our organization and partners.
* More information about what may be happening inside our network.
* Better awareness of security problems that might otherwise remain unnoticed.

From a leadership perspective, high-quality CTI can also help reduce business risk.

For example, when an organization collects and analyzes information about an attacker targeting its business, leadership can use that information to:

1. Understand the potential risk.
2. Prepare a contingency plan.
3. Decide what to do if an incident happens.
4. Communicate the problem clearly to the appropriate people.

<img width="577" height="159" alt="image" src="https://github.com/user-attachments/assets/1128835f-2111-449f-8942-2adb80de94f8" />

As raw information is collected, analyzed, and given context, it becomes **intelligence**.

This intelligence can be divided into three main categories:

1. **Strategic Intelligence**
2. **Operational Intelligence**
3. **Tactical Intelligence**

Each type is useful for different teams and different levels of an organization.

In the diagram below, the ideal situation is where these three types of intelligence overlap.

At this point, a CTI analyst can provide a more complete picture of the attacker and how they operate.

<img width="289" height="254" alt="image" src="https://github.com/user-attachments/assets/73f1a608-77ca-499d-a1b4-7e955ffc109a" />

---

## Strategic Intelligence

Strategic Intelligence is mainly designed for **high-level decision-makers**.

It is characterized by:

* Being used by C-suite executives, VPs, and other company leaders.
* Connecting threat information to business risks.
* Helping leaders make business and security decisions.
* Providing a broad view of an attacker's activities over time.
* Describing the attacker's TTPs and **Modus Operandi (MO)**.
* Trying to answer:

  * **Who?**
  * **Why?**

### Example

Imagine a strategic intelligence report about **APT28**, also known as **Fancy Bear**, which is publicly associated with Russian government-linked activity.

The report could explain:

* The group's previous campaigns.
* Its reported motivations, such as political espionage.
* The types of organizations it has targeted.
* Its long-term strategies.
* How its tools and techniques have changed over time.
* How historical activity and geopolitical events provide context for its behavior.

> 🎯 **Simple meaning:** Strategic Intelligence gives leaders the **big picture**.

---

## Operational Intelligence

Operational Intelligence provides more detail about **specific attacker campaigns and operations**.

It is characterized by:

* Including information about the attacker's TTPs.
* Providing information about specific adversary campaigns.
* Giving more detail than strategic intelligence.
* Often being used by mid-level management and security teams.
* Trying to answer:

  * **How?**
  * **Where?**

### Example

Imagine an operational intelligence report about a ransomware campaign conducted by the **REvil** group.

The report could explain:

* How the attackers gained initial access.

  * Phishing
  * Exploiting vulnerabilities
* How they moved through the network.

  * Credential dumping
  * Abusing Windows administrative tools
* How they delivered and executed the ransomware.
* When they typically carried out the attack.

For example, attackers might perform the final ransomware execution outside normal business hours to increase the potential disruption.

> 🎯 **Simple meaning:** Operational Intelligence explains **how an attacker carries out a campaign**.

---

## Tactical Intelligence

Tactical Intelligence provides **technical information that security defenders can use immediately**.

It is characterized by:

* Providing immediate, actionable information.
* Being given directly to network defenders.
* Including technical details about attacks that have happened or may happen soon.

### Example

A tactical intelligence report about REvil could contain:

* Malicious IP addresses.
* Malicious URLs.
* Malicious domains.
* File hashes of known ransomware samples.
* Specific file paths.
* Registry keys.
* Mutex names.
* Unique strings found in the ransomware code.

Security tools and incident responders can use this information to detect, block, or investigate the threat.

> 🎯 **Simple meaning:** Tactical Intelligence provides the **technical details defenders can use right now**.

---

## How The Three Types Overlap

These three types of intelligence are not completely separate.

There is some **overlap** between Strategic, Operational, and Tactical Intelligence.

That is why they are often shown using a **Venn diagram**.

Tactical Intelligence can help create a better understanding of an attacker's operations.

Operational Intelligence can contribute to the larger strategic picture.

At the same time, strategic and operational intelligence can provide context that helps analysts understand tactical information.

> 🔑 **Key idea:** The three types support each other rather than existing completely independently.

---

# How To Go Through A Tactical Threat Intelligence Report

Tactical threat intelligence reports often contain **Indicators of Compromise (IOCs)** and technical information.

For SOC analysts and Threat Hunters, reading these reports requires a structured process.

Let's use a theoretical example involving an **Emotet malware campaign**.

---

## 1. Comprehending the Report's Scope and Narrative

First, understand the **overall story and purpose of the report**.

Imagine the report describes an ongoing Emotet campaign targeting organizations in our industry.

The report might explain:

* Who the attackers are.
* What they want.
* What types of organizations they are targeting.
* How the campaign is being conducted.

Before looking at individual IP addresses or file hashes, we should first understand the bigger picture.

This helps us decide whether the threat is relevant to our organization.

> 💡 **Simple meaning:** First understand **what is happening and why** before looking at individual technical details.

---

## 2. Spotting and Classifying the IOCs

Tactical intelligence usually contains a list of **Indicators of Compromise (IOCs)**.

In our Emotet example, IOCs might include:

* IP addresses belonging to Command and Control (C2) servers.
* File hashes of Emotet malware.
* Email addresses used in phishing attacks.
* Phishing email subject lines.
* Malicious URLs.
* Registry changes made by the malware.

To make this information easier to work with, we can organize IOCs into categories.

### Network-Based IOCs

Examples:

* IP addresses
* Domains

### Host-Based IOCs

Examples:

* File hashes
* Registry keys
* File paths

### Email-Based IOCs

Examples:

* Email addresses
* Email subject lines

Other possible IOCs include:

* Mutex names created by malware.
* SSL certificate hashes.
* Specific API calls used by malware.
* Unusual network traffic patterns.
* Specific User-Agent strings.
* HTTP headers.
* DNS request patterns.

We can also add additional information to an IOC.

For example, an IP address might include:

* Geographic location.
* WHOIS information.
* Associated domains.

> 🔑 **Key idea:** Organizing IOCs makes them easier to understand, investigate, and use.

---

## 3. Comprehending the Attack's Lifecycle

The report will often describe the attacker's **Tactics, Techniques, and Procedures (TTPs)**.

These may be mapped to the **MITRE ATT&CK framework**.

For example, an Emotet campaign might follow a sequence such as:

1. **Initial Access** — The attacker sends a spear-phishing email.
2. **Execution** — The victim executes the malicious payload.
3. **Persistence** — The malware attempts to remain on the system.
4. **Defense Evasion** — The attacker attempts to avoid security detection.
5. **Command and Control** — The malware communicates with attacker-controlled infrastructure.
6. **Additional Actions** — The attacker may steal data or install additional malware.

Understanding this lifecycle helps defenders understand **what the attacker may do next**.

> 🔄 **Simple meaning:** Don't look at each attack step separately. Try to understand the **whole attack process**.

---

## 4. Analysis and Validation of IOCs

Not every IOC is equally useful or accurate.

Before using an IOC, analysts should **validate it**.

This can involve checking it against additional threat intelligence sources or databases, such as:

* VirusTotal
* AlienVault OTX
* Other trusted threat intelligence sources

### Check the Age of the IOC

An old IOC may no longer be useful.

For example, an attacker might have:

* Changed their IP address.
* Registered a new domain.
* Changed their malware.
* Changed their attack techniques.

### Check the Context

An IOC should not automatically be considered malicious without understanding its context.

For example, an IP address used by an attacker as a C2 server could also host legitimate websites, especially in shared cloud environments.

### Check the Source

Analysts should consider:

* Where did the IOC come from?
* Is the source reliable?
* Has this IOC been confirmed by other sources?
* Was the IOC previously added to a whitelist?

### Consider False Positives

A **false positive** happens when a security system reports something as suspicious or malicious when it is actually legitimate.

Too many false positives can cause **alert fatigue**, where analysts become overwhelmed by alerts and may have difficulty noticing genuinely important ones.

> ⚠️ **Key idea:** Don't blindly trust every IOC. **Validate it and understand its context first.**

---

## 5. Incorporating the IOCs into our Security Infrastructure

After validating an IOC, we can add it to our security systems.

For example:

| IOC                  | Possible Security Action       |
| -------------------- | ------------------------------ |
| Malicious IP address | Add it to firewall rules       |
| Malicious domain     | Block or monitor the domain    |
| File hash            | Add it to EDR detection        |
| Email address        | Update email security rules    |
| Malicious URL        | Block or monitor the URL       |
| Network behavior     | Create IDS/IPS detection rules |

For email-related IOCs, we may update:

* Email security gateways.
* Anti-spam systems.
* Email filtering rules.

### ⚠️ Consider Business Impact

Blocking something is not always the correct action.

For example, an IP address might appear suspicious but also be used by an important business service.

If we block it immediately, we could accidentally interrupt a critical service.

In this situation, **monitoring or alerting instead of blocking** may be more appropriate.

All changes should also:

* Be documented.
* Follow change-management procedures.
* Receive the required approval.
* Be tested when appropriate.

This helps protect system stability and prevents accidental disruption.

> 🔑 **Key idea:** Security actions should protect the organization **without unnecessarily breaking legitimate business operations**.

---

## 6. Proactive Threat Hunting

After learning about the threat, security analysts can actively search the organization's environment for signs of it.

For the Emotet example, analysts might:

* Search logs for connections to known C2 servers.
* Search endpoints for known malware file hashes.
* Search email logs for phishing indicators.
* Investigate suspicious PowerShell activity.

However, Threat Hunting should **not only search for known IOCs**.

Attackers can change their IP addresses, domains, files, and other indicators.

Therefore, analysts should also search for the **behavior and techniques** described in the report.

For example, if Emotet is known to use PowerShell, analysts can search for suspicious PowerShell activity even when the activity does not exactly match one of the known IOCs.

This can help detect **new or modified versions of the threat**.

> 🔎 **Key idea:** Hunt for both **known indicators** and **attacker behavior**.

---

## 7. Continuous Monitoring and Learning

After implementing the IOCs, the security team should continue monitoring the environment.

If a detection occurs, it should trigger the organization's predefined **incident response process**.

The team should also learn from the intelligence report and improve security controls.

For example, the organization might:

* Train users to recognize the phishing techniques used in the campaign.
* Improve detection rules.
* Improve monitoring.
* Add new security controls.
* Update incident response procedures.

### Share New Intelligence

Security teams can also contribute information back to the wider threat intelligence community.

If the organization discovers:

* New IOCs
* New TTPs
* New attacker behavior
* New infrastructure

that information can potentially be shared through appropriate threat intelligence platforms and **ISACs/ISAOs**.

**ISAC** means **Information Sharing and Analysis Center**.

**ISAO** means **Information Sharing and Analysis Organization**.

Sharing useful information can help other organizations defend themselves against the same threat.

> 🔄 **Key idea:** Threat intelligence is a continuous cycle: **learn → detect → investigate → improve → share → learn again**.

---

## Final Takeaway

This step-by-step process uses the Emotet example, but the same general approach can be used for many tactical threat intelligence reports containing IOCs.

The important thing is to be:

* **Systematic** — follow a clear process.
* **Comprehensive** — consider both technical indicators and attacker behavior.
* **Proactive** — don't wait for an attack to become obvious.
* **Careful** — validate information before taking action.
* **Continuous** — keep monitoring and learning from new intelligence.

> 🧠 **The main lesson:** A tactical threat intelligence report is not just a list of IP addresses, domains, hashes, and other IOCs. Its real value comes from understanding the **threat, validating the information, turning it into defensive actions, hunting for related activity, and continuously improving the organization's defenses.**
