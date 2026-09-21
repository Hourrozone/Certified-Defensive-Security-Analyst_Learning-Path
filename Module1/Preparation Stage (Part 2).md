## Preparation Stage (Part 2), Simplified

This part focuses on protecting against incidents before they happen. While prevention is not the incident handler's primary duty, understanding these controls helps you know what to look for during an investigation.

---

### 1. DMARC (Email Protection)

Most attacks start with phishing, making DMARC one of the best defenses against them.

#### The 3 Layers of Email Authentication:

* **SPF (Sender Policy Framework):** Checks if the email came from an authorized mail server for that domain (e.g., verifies an email claiming to be from `@yourbank.com` came from an official mail server).
* **DKIM (DomainKeys Identified Mail):** Attaches a digital signature to the email to confirm it was not tampered with in transit.
* **DMARC:** Sits on top of SPF and DKIM. It instructs receiving mail servers how to handle failed checks (allow, quarantine, or reject).

## How Email Authentication Works (SPF ➔ DKIM ➔ DMARC)

Think of sending an email like sending a physical letter.

---

##### 📩 SPF — "Is this mailman allowed to deliver for this address?"

SPF is a public list published by your organization stating which mail servers are permitted to send emails on behalf of your domain.

When a receiving server gets an email claiming to be from `@yourbank.com`, it checks if the sending server matches your bank's approved list.

* **Pass:** Came from an approved server.
* **Fail:** Came from an unauthorized server.

> 🛡️ **What it catches:** Attackers sending emails from their own servers while pretending to be your domain.  
> ⚠️ **What it misses:** Legitimate servers being abused, or email content being modified in transit.

---

##### 🔏 DKIM — "Was this letter tampered with after it was sent?"

DKIM adds a digital signature to every outgoing email, acting like a wax seal on an envelope.

The receiving server checks the signature to verify its authenticity and integrity.

* **Pass:** Signature is valid and content remains unchanged.
* **Fail:** Signature is missing or content was modified.

> 🛡️ **What it catches:** Email content tampered with in transit, and emails forged without access to your signing key.  
> ⚠️ **What it misses:** Enforcement actions. DKIM only reports the result, it does not decide what to do with failed emails.

---

##### 🛡️ DMARC — "What should we DO when SPF or DKIM fails?"

DMARC is the policy enforcement layer sitting on top of both SPF and DKIM. It tells receiving servers how to handle emails claiming to be from your domain that fail checks.

###### Three DMARC Policies:

| Policy | Action |
| :--- | :--- |
| **`none`** | Do nothing, monitor and log reports only |
| **`quarantine`** | Send the email directly to spam/junk folders |
| **`reject`** | Block the email completely, preventing delivery |

> 🌐 **Real-World Example:** An attacker sends a phishing email pretending to be your CEO. It fails SPF (wrong sending server), and your DMARC policy is set to `reject`. The email is blocked before it ever reaches the employee's inbox.

---

##### 🔄 How All Three Work Together

> 🚨 **Critical Warning:** DMARC requires thorough testing before enabling `reject` mode. Misconfigurations can silently block legitimate emails with no way to recover them.

* **SOC Impact:** DMARC rejects spoofed emails (e.g., impersonating an executive) before reaching recipients.
* **Advanced Usage:** Header filtering can flag or block external emails failing DMARC. Watch for false positives from third-party marketing tools due to domain mismatches.

---

### 2. Endpoint Hardening + EDR

Endpoints (laptops, workstations) are where most attacks land first. Key hardening actions:

* **Disable LLMNR/NetBIOS:** Prevents attackers from abusing legacy protocols to capture network credentials.
* **Implement LAPS:** Assigns each machine a unique local admin password so one stolen password cannot unlock others. Remove local admin rights from regular users.
* **Restrict or Disable PowerShell:** Set to `ConstrainedLanguage` mode to limit malicious script capabilities.
* **Enable Attack Surface Reduction (ASR) Rules:** Block common attack behaviors in Microsoft Defender.
* **Block Execution from User-Writable Folders:** Prevent executable scripts (`.hta`, `.vbs`, `.cmd`, `.bat`, `.js`) from launching in `Downloads`, `Desktop`, or `AppData`.
* **Watch for LOLBins (Living Off the Land Binaries):** Monitor legitimate Windows tools (e.g., `certutil`, `mshta`, `wscript`) abused by attackers to bypass whitelisting.
* **Host-Based Firewalls:** Block workstation-to-workstation communication and outbound traffic to LOLBins.
* **Deploy EDR with AMSI Support:** Enables security products to inspect obfuscated/encoded scripts in memory prior to execution.

---

### 3. Network Protection

* **Network Segmentation:** Isolate critical systems so a breach in one zone cannot spread across the environment.
* **No Direct Internet Exposure:** Place required external-facing resources in a DMZ (isolated network zone).
* **IDS/IPS with SSL/TLS Inspection:** Inspect actual decrypted traffic content rather than relying strictly on IP reputation.
* **802.1X Authentication:** Restrict corporate network connections to approved devices only, blocking unauthorized or BYOD endpoints.
* **Conditional Access (Entra ID / Azure):** Restrict cloud access strictly to company-managed devices.

---

### 4. Identity & Access Management (PIM / MFA / Passwords)

Stealing privileged credentials is the primary escalation path in Active Directory environments.

* **Avoid Common Mistakes:** Weak admin passwords (e.g., `Password1!`) or admin accounts sharing passwords with regular user accounts.
* **Use Passphrases:** Long, easy to remember, and complex (e.g., `i LIK3 my coffeE warm`).
* **Enforce MFA:** Mandatory for all administrative access without exception.

---

### 5. Vulnerability Scanning

* Run continuous vulnerability scans across the entire environment.
* Remediate all High and Critical findings manually.
* If a vulnerable system cannot be patched immediately, isolate it from the network.

---

### 6. User Awareness Training

Users are the most targeted entry point. Ongoing training and testing significantly reduce successful attacks:

* Monthly simulated phishing campaigns.
* USB drop tests around physical facilities.

---

### 7. Active Directory (AD) Security Assessment

Active Directory is the backbone of most enterprise environments and contains well-documented attack paths.

* Conduct regular internal or third-party AD assessments to find misconfigurations and escalation paths early.
* **Goal:** Prevent a single compromised endpoint from providing a direct path to Domain Admin.

---

### 8. Purple Team Exercises

* **Red Team:** Offensive security team simulating real-world attacks.
* **Blue Team:** Defensive team (SOC, incident response) detecting and responding.
* **Purple Team Exercise:** Collaborative review where the Red Team shares exact execution paths to evaluate what the Blue Team detected or missed.

> 🎯 **Key Takeaway:** Use undetected attacks as improvement opportunities to refine logging, detection rules, and IR playbooks.
