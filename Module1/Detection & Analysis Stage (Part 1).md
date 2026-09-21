## Detection & Analysis Stage (Part 1), Simplified

This stage focuses on identifying incidents and understanding what is occurring before taking action. Detection originates from multiple sources, and analysis involves gathering sufficient context to make informed decisions.

---

### Where Detections Come From

* An employee notices suspicious activity.
* An automated alert triggers from a security tool (EDR, IDS, Firewall, SIEM).
* Proactive threat hunting activities.
* A third party (external company, vendor, or partner) notifies you of a compromise.

---

### Build Detection at Every Layer

Do not rely on a single point of detection. Comprehensive coverage requires defense-in-depth:

| Layer | Tools Used |
| :--- | :--- |
| **Network Perimeter** | Firewalls, IDS/IPS, DMZ |
| **Internal Network** | Local firewalls, Host-based IDS/IPS |
| **Endpoint** | Antivirus, EDR |
| **Application** | Application logs, Service logs |

> 🛡️ **Key Takeaway:** The more layers covered, the harder it becomes for an attacker to move undetected across the entire environment.

---

### Initial Investigation — Gather Context First

Before declaring a full incident response, gather enough data to understand the scope. Acting without context leads to inaccurate conclusions and flawed response steps.

> 🌐 **Example:** An alert shows `"Admin account connected to IP 10.0.0.5 at 13:31"`. Determining whether this is suspicious requires knowing what system lives at that IP, the timestamp's timezone, and if that admin account routinely performs this task.

#### Context Checklist:

* **Basic Info:** When was the incident reported, detected, and by whom? How was it detected?
* **Type & Scope:** What type of incident is it (phishing, ransomware, system outage)? Which systems are impacted?
* **Attacker Actions:** Who accessed those systems and what actions were performed? Is the attack ongoing?
* **System Details:** Physical location, OS, IP address, hostname, owner, purpose, and current operational state.
* **Malware Indicators:** Detection timestamp, malware classification, affected systems, file hashes, and safe copies of malicious files.

> 🎯 **Context Matters:** A compromised CEO laptop requires a significantly different handling protocol than a compromised intern workstation.

---

### Build an Incident Timeline

Organize evidence into a chronological timeline sorted by when events actually occurred, rather than the order they were discovered.

* Keeps the investigation structured and readable.
* Provides a complete picture of attacker activities across systems.
* Helps determine if newly discovered evidence belongs to the active incident or is unrelated.
* Reveals true **"patient zero"** (e.g., an attack thought to have started Monday might trace back to a payload placed two weeks earlier).

#### Timeline Format Example:

<img width="900" height="159" alt="image" src="https://github.com/user-attachments/assets/0d736086-ed00-4469-af87-1546c1c2ac18" />

Focus timeline entries strictly on attacker behavior: lateral movement, new connections, file downloads, and tool executions.

<img width="2406" height="911" alt="image" src="https://github.com/user-attachments/assets/3a16ea90-e850-4baa-9d58-9dc3876913e1" />

---

### Assess Severity and Scope

Answer these questions to evaluate the overall severity:

* What is the business impact of the exploit?
* What prerequisites or privileges does the attacker need to pull it off?
* Can business-critical systems or core infrastructure be affected?
* Are there official remediation steps or vendor patches available?
* How many total systems are impacted?
* Is this exploit actively used in wild campaigns?
* Does the payload spread autonomously (worm-like behavior)?

> 🚨 **Escalation Trigger:** A self-spreading exploit used in active campaigns, high business impact, or a large number of affected systems demands immediate escalation.

---

### Confidentiality & Communication Rules

Treat all incident information on a strict need-to-know basis unless legal mandates or executive leadership direct otherwise.

* **Insider Risk:** The attacker could be an internal employee.
* **Legal Liability:** Uncontrolled breach disclosures create legal exposure.
* **External Media:** All external communications must route strictly through designated PR and legal teams.

#### Setting Expectations:

When opening an investigation, define clear parameters for stakeholders:

* Type of incident under review.
* Available evidence sources.
* Estimated investigation timeframe.
* Feasibility of identifying the threat actor.

Keep leadership updated as findings evolve. While scope shifts are normal during an investigation, going silent is unacceptable.

---

### 🐝 Case Management Tools (e.g., TheHive)

Dedicated incident response platforms like TheHive organize security workflows. When an alert arrives, analysts can:

* Assign alerts to specific team members.
* Promote alerts to full investigation cases.
* Document investigation details, timelines, and raw evidence.
* Tag findings with MITRE ATT&CK techniques (e.g., `T1003.001` - LSASS dumping).
* Close and archive cases with audit trails once resolved.

> 📌 **Key Takeaway:** Centralized case tracking keeps investigations organized, auditable, and collaborative when multiple analysts work the same incident.
