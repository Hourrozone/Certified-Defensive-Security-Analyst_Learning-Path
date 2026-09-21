## Preparation Stage (Part 1), Simplified

Preparation has two primary goals:

* **Build your incident handling capability:** Be ready to respond when an incident occurs.
* **Prevent incidents from happening:** Harden systems before attackers get in.

> 💡 **Note:** Preventing incidents isn't the incident handling team's direct job, but it directly affects how well they can perform their role.

---

### What You Need Before an Incident Happens

* A skilled incident handling team (can be partially outsourced, but some in-house capability is mandatory).
* A security-aware workforce.
* Clear written policies and documentation.
* The right tools (software + hardware).

---

### Critical Documentation to Have Ready

During a real incident, you will not have time to figure these out from scratch.

* **Contact lists:** Incident team, legal/compliance, management, IT support, PR/media, law enforcement, ISPs, and external IR teams.
* **Incident response policy, plan, and procedures.**
* **Information sharing policy:** What can be shared, with whom, and when.
* **System and network baselines:** What "normal/clean" looks like (golden images).
* **Network diagrams.**
* **Asset management database:** Comprehensive inventory of all systems in the organization.
* **Pre-approved privileged accounts:** Enabled only when an incident is confirmed, disabled immediately after, with mandatory password reset.
* **Emergency purchasing authority:** Ability to buy tools quickly (up to a set amount) without a full procurement process. Waiting weeks for a $500 tool approval during an active incident is a major roadblock.
* **Forensic/investigative cheat sheets.**

---

### Legal & Compliance

Some incidents carry strict legal reporting requirements.

> 🌐 **Example:** A data breach involving customer data must be reported to law enforcement within a specific time window under GDPR.

Requirements vary by location and type of incident, always consult your legal and compliance team before and during an incident.

---

### Document the Incident As It Happens

Having pre-written docs isn't enough, you must record what you are doing in real time. Incidents are fast-moving, so forcing the habit of note-taking is essential.

Every note should answer: **Who, What, When, Where, Why, and How.**

#### Required Entry Elements:

* Timestamp
* Action performed
* Result of that action
* Who did it

---

### Essential IR Toolsets

#### 💻 Digital & Forensic
* **Forensic workstation:** Dedicated laptop per team member (antivirus disabled, used for malware analysis and disk imaging).
* **Disk imaging tools**
* **Memory capture and analysis tools**
* **Live response tools**
* **Log analysis tools**
* **Network capture and analysis tools**
* **IOC creator + org-wide search capability:** Create and search Indicators of Compromise.

#### 🛠️ Physical
* Network cables and switches
* **Write blockers:** Prevent accidental modification of evidence drives.
* Hard drives for forensic imaging
* Power cables
* Screwdrivers, tweezers, and hardware tools

#### 📋 Administrative
* **Chain of custody forms:** Tracks who handled evidence and when (critical for legal cases).
* Encryption software
* Ticket tracking system
* Secure storage facility

---

### 🎒 The Jump Bag

A jump bag is a pre-packed kit containing all essential tools, always ready to grab and go. Without it, assembling everything during an active incident could delay response times by days or weeks.

---
<img width="1138" height="567" alt="image" src="https://github.com/user-attachments/assets/cc5c60ab-ce8f-4f66-b400-960606dd527d" />

### 🚨 Two Rules That Cannot Be Broken

1. **Keep your documentation system completely separate from organizational infrastructure:** Assume your entire domain is compromised. If your IR notes live on a company server and that server goes down, or the attacker accesses it, you lose everything.
2. **Communicate about the incident through external channels only:** Never use company email or internal chat tools to discuss an active incident. Assume the attacker can read internal communications.

> 📌 **Key Takeaway:** Treat your IR communication and documentation as if the attacker already has full access, because they very well might.
