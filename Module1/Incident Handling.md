## Incident Handling Module

### Incident Handling Definition & Scope

An event is an action occurring in a system or network. Examples of events include:
- A user sending an email.
- A mouse click.
- A firewall allowing a connection request.

<img width="1382" height="657" alt="image" src="https://github.com/user-attachments/assets/c18848b6-fd91-45f7-a6e6-a256a1f9ec4c" />

---

### Different Types of Real-World Incidents

#### 🚨 Leaked Credentials

* **Colonial Pipeline Ransomware Attack:** The Colonial Pipeline, a major American oil pipeline system, fell victim to a ransomware attack. This attack originated from a breached employee's personal password, likely found on the dark web, rather than a direct attack on the company's network. The attackers gained access to the company's systems using a compromised password for an inactive VPN (Virtual Private Network) account, which did not have Multi-Factor Authentication (MFA) enabled.

#### 🚨 Default / Weak Credentials

* **Mirai Botnet (2016):** The Mirai botnet scanned for IoT devices using factory or default credentials (e.g., `admin/admin`) and conscripted them into a massive DDoS botnet. This led to large-scale DDoS disruptions affecting companies like Dyn and OVH, with hundreds of thousands of devices infected. The root cause was the devices being shipped with unchanged default credentials and poor remote access security.
* **LogicMonitor Incident (2023):** Some LogicMonitor customers were compromised because the vendor issued weak default passwords to customer accounts. Affected customers experienced follow-on ransomware incidents or unauthorized access. The root cause involved vendor-assigned weak/default credentials and delayed enforcement of password hardening.

#### 🚨 Outdated Software / Unpatched Systems

* **Equifax (2017) Breach:** Attackers exploited a known Apache Struts vulnerability (CVE-2017-5638) in Equifax’s web application. This breach exposed the personal data of approximately 143–147 million people, leading to major regulatory and legal fallout. The incident occurred due to a failure to apply a publicly released patch in a timely manner.
* **WannaCry (2017):** The WannaCry ransomware spread as a worm using the SMB EternalBlue exploit, affecting more than 200,000 systems across over 150 countries. High-profile impacts included hospitals and enterprises. This incident was due to unpatched Windows systems, despite the MS17-010 patch being available before the outbreak.

#### 🚨 Rogue Employee / Insider Threat

* **Cash App / Block Inc. (2021 Disclosure; Public 2022 Notice):** A former employee accessed the personal information of millions of Cash App users, as reported in company disclosures. Approximately 8.2 million current and former customers were potentially impacted, leading to regulatory scrutiny and settlements. The root cause was the abuse of legitimate employee access and insufficient internal controls and monitoring.

#### 🚨 Phishing / Social Engineering

* **Industry Trend & Representative Data:** Phishing is a pervasive vector used to obtain credentials, deliver malware, or trick users into enabling remote access. It frequently leads to account compromise, fraud, and network footholds. A significant portion of breaches over multiple years are linked to phishing.
* **U.S. Interior Department Phishing Attack:** Attackers used an "evil twin" technique to trick individuals into connecting to a fake Wi-Fi network, allowing hackers to steal credentials and access the network. This incident revealed a lack of secure wireless network infrastructure and insufficient security measures, including weak user authentication and inadequate network testing.
* **2020 Twitter Account Hijacking:** In 2020, many high-profile Twitter accounts were compromised by outside parties to promote a bitcoin scam. Attackers gained access to Twitter's administrative tools, allowing them to alter accounts and post tweets directly. They appeared to have used social engineering to gain access to the tools via Twitter employees.

#### 🚨 Supply-Chain Attack

* **SolarWinds Orion (2020):** Nation-state actors compromised the SolarWinds build/release environment and injected a malicious backdoor into Orion updates, which were distributed to thousands of customers. This caused wide-reaching espionage and unauthorized access across government and private sectors, leading to protracted detection and remediation efforts.

---

### Standard Incident Reports

* [DFIR Report - Confluence Exploit Leads to LockBit Ransomware](https://thedfirreport.com/2025/02/24/confluence-exploit-leads-to-lockbit-ransomware/)
* [Cybereason Research - Chaes E-Commerce Malware (PDF)](https://www.cybereason.com/hubfs/dam/collateral/reports/11-2020-Chaes-e-commerce-malware-research.pdf)
* [Palo Alto Unit 42 - Global Incident Response Report](https://www.paloaltonetworks.com/engage/unit42-2025-global-incident-response-report)
