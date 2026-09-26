# 🛡️ The Threat Hunting Process

Threat hunting is not just searching randomly for suspicious activity.

A good threat hunt follows a process.

The main process is:

```text
1. Set the Stage
       ↓
2. Create a Hypothesis
       ↓
3. Design the Hunt
       ↓
4. Gather & Examine Data
       ↓
5. Evaluate Findings
       ↓
6. Mitigate the Threat
       ↓
7. After the Hunt
       ↓
8. Continuous Learning
```

---

# 1. 🛠️ Setting the Stage

This is the **planning and preparation** phase.

Before starting a hunt, we need to understand:

* What we are looking for
* What systems are important
* What threats the organization may face
* What attackers are currently doing
* What security data is available

We also need to make sure our environment is ready for hunting.

For example, we should have enough logging and properly configured security tools.

Common tools include:

* SIEM
* EDR
* IDS
* Network monitoring
* Endpoint logging

---

## 🔍 Example

Before starting a hunt, the threat hunting team may:

1. Read recent threat intelligence reports.
2. Study vulnerabilities affecting the organization.
3. Study attacker TTPs.
4. Identify important systems and assets.
5. Enable logging on servers and endpoints.
6. Configure SIEM, EDR, and IDS.
7. Monitor new threat intelligence.
8. Learn about new attacker techniques.

For example:

```text
Threat Intelligence
        ↓
Understand the threat
        ↓
Identify important assets
        ↓
Make sure logs are available
        ↓
Configure security tools
        ↓
Ready to Hunt
```

### 💡 Simple idea

> **Before hunting, make sure you know what you are hunting for and have the data needed to find it.**

---

# 2. 🧠 Formulating Hypotheses

After preparation, we create a **hypothesis**.

A hypothesis is simply an **idea about what might be happening** that we want to investigate.

It can come from:

* Threat intelligence
* Security alerts
* Recent attacks
* Vulnerabilities
* Industry information
* The hunter's experience

The hypothesis should be **specific and testable**.

---

## 🔍 Example

Suppose threat intelligence says an attacker is exploiting a web server vulnerability.

We can create this hypothesis:

> **"An attacker may be using the known vulnerability to gain access to our web server."**

Now we know what we need to investigate.

Another example:

> **"An attacker may be using phishing emails to gain access to employee accounts."**

The important thing is that we should be able to search our data and determine whether the hypothesis is true or false.

---

# 3. 🛠️ Designing the Hunt

Once we have a hypothesis, we need to decide **how we are going to test it**.

We need to identify:

* What data we need
* Which logs we should check
* Which tools we should use
* What IoCs we should search for
* What suspicious patterns we should look for
* What queries or scripts we should create

---

## 🔍 Example

Suppose our hypothesis is:

> **"An attacker may be using a compromised web server to communicate with a C2 server."**

We might investigate:

* Web server logs
* Network logs
* DNS logs
* Endpoint logs
* Firewall logs

We can then create SIEM queries to search for:

* Suspicious IP addresses
* Suspicious domains
* Unusual connections
* Known malicious hashes
* Unusual processes

---

## 🧠 Simple idea

Think of it like planning an investigation:

```text
Hypothesis
    ↓
What evidence do I need?
    ↓
Where can I find that evidence?
    ↓
What tools can I use?
    ↓
What exactly should I search for?
```

---

# 4. 🔍 Data Gathering and Examination

This is where the **actual hunting begins**.

We collect and analyze the data we identified during the previous step.

Possible data includes:

* Windows Event Logs
* Sysmon logs
* Network traffic
* DNS logs
* Endpoint data
* Web server logs
* Firewall logs
* Email logs

The goal is to find evidence that either:

> **Supports the hypothesis**

or

> **Disproves the hypothesis**

---

## 🔍 Example

Suppose we are hunting for suspicious communication.

We might:

```text
Collect network logs
        ↓
Search external connections
        ↓
Find unusual domain
        ↓
Check domain reputation
        ↓
Check endpoint activity
        ↓
Look for related processes
```

We may use tools such as:

* Log analyzers
* Packet analyzers
* Malware sandboxes
* SIEM
* EDR

---

## 🔄 Hunting Can Change

Threat hunting is not always a straight process.

While investigating, we may discover something new.

For example:

```text
Original hypothesis
        ↓
Search logs
        ↓
Find unexpected activity
        ↓
New information
        ↓
Change or improve hypothesis
        ↓
Continue hunting
```

So, the hunter may need to change the investigation based on what the data shows.

---

# 5. 🧠 Evaluating Findings and Testing the Hypothesis

After analyzing the data, we need to understand what we found.

We need to determine:

* Did we confirm the hypothesis?
* Did we disprove it?
* What did the attacker do?
* Which systems are affected?
* How serious is the situation?
* What should happen next?

---

## 🔍 Example

Suppose our hypothesis was:

> **"An attacker is trying to brute-force user accounts."**

During the investigation, we find:

```text
Many failed logins
        ↓
Same source IP
        ↓
Known malicious IP
        ↓
Multiple user accounts targeted
```

This provides evidence supporting the hypothesis.

The team can then investigate further.

---

## Another Example

Suppose we suspected C2 communication.

We find:

```text
Internal computer
       ↓
Connection
       ↓
Known malicious domain
```

This may support the hypothesis that the system is communicating with a C2 server.

The team then needs to determine:

* Which computer is infected?
* Which user was using it?
* What process made the connection?
* When did it start?
* What other systems are involved?

---

# 6. 🚨 Mitigating Threats

If the threat is confirmed, the organization needs to take action.

This can include:

* Isolating affected systems
* Removing malware
* Blocking malicious domains/IPs
* Patching vulnerabilities
* Changing configurations
* Strengthening security controls

The goal is to:

> **Remove the threat and reduce further damage.**

---

## 🔍 Example

Suppose a computer is communicating with a malicious C2 server.

The team may:

```text
Compromised Computer
        ↓
Isolate from network
        ↓
Investigate
        ↓
Remove malware
        ↓
Patch vulnerability
        ↓
Block C2 communication
        ↓
Restore system
```

---

# 7. 📋 After the Hunt

The hunt does not end when we find the threat.

After the investigation, we should document what happened.

We should record:

* What we searched
* What data we used
* What we found
* What hypothesis we tested
* What tools we used
* What actions were taken
* What the final result was

---

## 🛠️ Improve Security After the Hunt

The findings can then be used to improve security.

For example:

### Improve Detection Rules

If we discovered a suspicious PowerShell command, we may create or improve a SIEM detection rule for it.

### Update Threat Intelligence

If we discovered a new malicious IP or hash, we can add it to threat intelligence.

### Improve Incident Response

We can update the incident response playbook based on what we learned.

### Improve Security Policies

We can change security procedures to reduce the chance of the same attack happening again.

---

## 💡 Simple idea

> **Every threat hunt should make the next hunt better.**

---

# 8. 🔄 Continuous Learning and Enhancement

Threat hunting is **not a one-time activity**.

The threat landscape changes constantly.

Attackers develop:

* New techniques
* New malware
* New tools
* New attack methods

Therefore, threat hunters need to continuously learn and improve.

After every hunt, the team should ask:

> What worked?

> What did not work?

> What did we miss?

> What can we improve?

---

## 🔍 Example

After completing a hunt, the team may discover that their existing searches were not good enough to detect a sophisticated attack.

They may then improve their hunting by using:

* Behavioral analysis
* Better queries
* New threat intelligence
* Machine learning
* New security tools

The team can also:

* Attend security training
* Follow security research
* Attend conferences
* Share knowledge with other threat hunters

---

# 🧠 Threat Hunting = Art + Science

Threat hunting requires both technical knowledge and creativity.

### Science

We use:

* Logs
* Data
* Queries
* Statistics
* Threat intelligence
* Security tools

### Art

We also need:

* Experience
* Creativity
* Critical thinking
* Understanding of attacker behavior
* Ability to think like an attacker

A good threat hunter understands both:

```text
Our Environment
       +
Attacker Behavior
       +
Security Data
       ↓
Better Threat Hunting
```

---

# 9. 🦠 The Threat Hunting Process vs. Emotet

Now let's apply the threat hunting process to **Emotet** malware.

The example shows how the same eight steps can be used to investigate a specific threat.

---

# Step 1 — 🛠️ Setting the Stage

First, the threat hunting team learns about Emotet.

They study:

* Previous Emotet attacks
* Emotet malware samples
* Threat intelligence reports
* Emotet TTPs
* Emotet infection methods

The team learns that Emotet can use methods such as:

* Malicious email attachments
* Malicious links
* Exploiting vulnerabilities

The team also identifies systems that may be important targets.

For example:

* User endpoints
* Systems with administrative privileges
* Email servers

---

# Step 2 — 🧠 Formulating Hypotheses

Next, the team creates a hypothesis based on what they know about Emotet.

For example:

> **"Emotet may be using compromised email accounts to send phishing emails containing malicious Word documents."**

This is specific enough to investigate.

The hypothesis could come from:

* Recent Emotet threat intelligence
* Previous Emotet campaigns
* Suspicious email security alerts

---

# Step 3 — 🛠️ Designing the Hunt

Now the team decides what data they need.

They may investigate:

* Email server logs
* Network logs
* Endpoint logs
* Malware samples

They create searches for Emotet-related activity.

For example, they may search for:

* Suspicious email subjects
* Suspicious attachments
* Malicious domains
* Known Emotet IP addresses
* Known file hashes
* Suspicious network connections

---

# Step 4 — 🔍 Data Gathering and Examination

Now the actual hunt begins.

The team collects and analyzes data.

For example:

```text
Email Logs
    ↓
Find suspicious attachment
    ↓
Check attachment
    ↓
Check endpoint
    ↓
Check network connections
    ↓
Look for Emotet activity
```

They may investigate:

* Email headers
* Network traffic
* Endpoint behavior
* Known Emotet IoCs

They may use:

* Email forensic tools
* Network packet analyzers
* Malware sandboxes

---

# Step 5 — 🧠 Evaluate Findings

Now the team asks:

> **"Did we find evidence that supports our hypothesis?"**

For example, they may discover:

```text
Multiple suspicious emails
        ↓
Similar subjects
        ↓
Similar attachments
        ↓
Known Emotet characteristics
```

This supports the hypothesis that Emotet-related phishing activity may be happening.

They may also discover:

```text
Internal Computer
       ↓
Connection
       ↓
Known Emotet C2 Server
```

This could support the possibility of an active Emotet infection.

The team then investigates:

* Which systems are affected?
* How many systems are affected?
* How did the infection happen?
* What did Emotet do?
* What is the potential impact?

---

# Step 6 — 🚨 Mitigating the Threat

If Emotet infections are confirmed, the team takes action.

They may:

### 1. Isolate infected systems

```text
Infected Computer
       ↓
Disconnect from network
```

This helps prevent the malware from spreading further.

### 2. Remove the malware

Endpoint security tools can be used to detect and remove Emotet.

### 3. Investigate compromised email accounts

The team checks whether email accounts were accessed or abused.

### 4. Patch vulnerabilities

If Emotet used a vulnerability, the organization should patch or otherwise mitigate it.

### 5. Block malicious communication

The organization can block known:

* Emotet C2 servers
* Malicious domains
* Other related indicators

---

# Step 7 — 📋 After the Hunt

After the Emotet investigation is complete, the team documents the results.

They may:

* Record newly discovered IoCs
* Update threat intelligence
* Improve SIEM detection rules
* Update incident response playbooks
* Improve security policies
* Update training

For example:

```text
Emotet Hunt
     ↓
New malicious IP discovered
     ↓
Add IoC to Threat Intelligence
     ↓
Create SIEM detection rule
     ↓
Improve future detection
```

---

# Step 8 — 🔄 Continuous Learning

The team should continue learning about Emotet because its behavior can change.

After each hunt, they review:

* Was our hypothesis useful?
* Were our queries effective?
* Did we miss anything?
* What new IoCs did we discover?
* What new techniques did Emotet use?

They can then improve their hunting methods.

For example, they may use:

* Behavior-based detection
* Machine learning
* New threat intelligence
* Improved hunting queries

They can also learn from:

* Security conferences
* Training
* Other threat hunting teams
* Security research

---

# 📌 Complete Threat Hunting Process

You can remember the process like this:

```text
🛠️ 1. SET THE STAGE
   Prepare the environment
   Understand threats and assets
          ↓
🧠 2. CREATE HYPOTHESIS
   What do we think might be happening?
          ↓
🛠️ 3. DESIGN THE HUNT
   Decide what data and tools to use
          ↓
🔍 4. GATHER & EXAMINE DATA
   Search logs and security data
          ↓
🧠 5. EVALUATE FINDINGS
   Confirm or reject the hypothesis
          ↓
🚨 6. MITIGATE
   Contain and remove the threat
          ↓
📋 7. AFTER THE HUNT
   Document and improve detections
          ↓
🔄 8. CONTINUOUS LEARNING
   Learn and improve the next hunt
          ↓
       START AGAIN
```

# 💡 Key Takeaway

The most important idea is:

> **Threat hunting is a continuous cycle, not a one-time investigation.**

A simple way to remember it:

**Prepare → Hypothesize → Hunt → Investigate → Respond → Learn → Improve → Hunt again**

And when using a real threat such as Emotet:

**Learn about Emotet → Create an Emotet hypothesis → Search logs/data → Find evidence → Confirm infection → Mitigate → Document → Improve future detection.**
