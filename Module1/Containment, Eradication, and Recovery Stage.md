# Containment, Eradication, and Recovery Stage

After the investigation is complete, the security team should understand:

* What type of incident happened
* How the attacker entered
* Which systems were affected
* What damage was caused
* What evidence was found

This information helps the team decide what to do next.

The next stages are:

```text
Investigation
     ↓
Containment
     ↓
Eradication
     ↓
Recovery
     ↓
Normal Operation
```

> 📌 **Main goal:** Stop the incident, remove the threat, and safely return the affected systems to normal operation.

---

<img width="1138" height="567" alt="image" src="https://github.com/user-attachments/assets/a1e1f115-5d25-4a33-9c60-8a388ebb1aae" />

# 🛑 1. Containment

**Containment** means taking action to **stop the incident from spreading or causing more damage**.

For example, if malware has infected several computers, the security team may isolate those computers from the network.

Containment is usually divided into:

* **Short-term containment**
* **Long-term containment**

> ⚠️ **Important:** Containment actions should be carefully planned and coordinated. If actions are taken too early or without a plan, the attacker may notice that they have been discovered and change their behavior.

---

## 🟡 Short-Term Containment

Short-term containment focuses on **quick actions that limit the damage** while trying to change the affected systems as little as possible.

Common actions include:

### 1. 🌐 Isolate the System

Move the affected computer to a separate or isolated **VLAN**.

A VLAN is a way to separate network devices into different network groups.

```text
Normal Network
      │
      ├── PC 1
      ├── PC 2
      ├── PC 3
      │
      └── 🚨 Infected PC
               │
               ▼
        Isolated VLAN
```

The infected system can then be investigated without allowing it to freely communicate with other systems.

---

### 2. 🔌 Disconnect the Network

Another simple option is to disconnect the network cable from the affected computer.

For wireless systems, the network connection can also be disabled.

> ⚠️ **Warning:** Disconnecting a system can change or destroy some evidence, so investigators should consider evidence collection before taking this action when possible.

---

### 3. 🌐 Block the Attacker's C2 Communication

Malware may communicate with an attacker's **Command and Control (C2)** server.

The security team can block or redirect the communication to prevent the malware from receiving commands.

```text
Malware
   │
   │ C2 communication
   ▼
Attacker's C2 Server
```

After containment:

```text
Malware
   │
   │ blocked / redirected
   ✕
   ▼
C2 Server
```

> 🧠 **C2 (Command and Control):** A system controlled by an attacker that can send commands to compromised computers.

---

## 📦 Preserve Evidence

Short-term containment also gives the security team time to prepare a better remediation plan.

Because the affected systems are changed as little as possible, investigators may still have an opportunity to collect evidence such as:

* Disk images
* Memory captures
* Malware samples
* Logs
* Network information

This evidence can be important for understanding what happened.

> 📌 **Backup:** In this context, the containment process may also include preserving or backing up forensic evidence before making major changes to the affected system.

---

## ⚠️ Before Shutting Down a System

Sometimes containment requires shutting down a computer.

Before doing this, the security team should:

1. Consider whether important evidence is still in memory.
2. Communicate the action to the business.
3. Get the required permission.
4. Collect important evidence first when possible.

> 🚨 **Critical:** Shutting down a computer can cause information stored only in RAM to be lost.

---

# 🔵 Long-Term Containment

Long-term containment focuses on **more permanent changes** that help keep the attacker from returning.

Examples include:

* 🔑 Changing user passwords
* 🔥 Adding or changing firewall rules
* 🛡️ Installing or enabling Host Intrusion Detection Systems (HIDS)
* 🔄 Applying security patches
* 📴 Shutting down affected systems when necessary
* 🔒 Improving system security settings

### Example

If an attacker obtained a user's password:

```text
Attacker
   ↓
Stolen Password
   ↓
User Account
   ↓
Internal System
```

A long-term containment action could be:

```text
Reset Password
      +
Enable MFA
      +
Review Account Activity
      ↓
Reduce Risk of Re-entry
```

> 💡 **Important:** The business and relevant teams should be kept informed while these changes are being made.

---

## ⚠️ Containment Does Not Mean the Incident Is Over

A common mistake is thinking:

> "The system is patched, so the incident is finished."

That is not necessarily true.

A patch may fix one vulnerability, but the attacker may have already:

* Created another account
* Installed malware
* Added persistence
* Stolen credentials
* Compromised another system

The following stages are still required:

```text
Containment
     ↓
Eradication
     ↓
Recovery
     ↓
Post-Incident Activities
```

---

# 🧹 2. Eradication

**Eradication** means **removing the attacker and the cause of the incident from the environment**.

After containment has stopped the immediate spread, the security team works to completely remove the threat.

Common eradication activities include:

* 🦠 Removing detected malware
* 💻 Rebuilding compromised systems
* 💾 Restoring systems from clean backups
* 🔄 Applying additional security patches
* 🔒 Hardening systems
* 🛡️ Improving security controls across the network

---

## 🦠 Remove the Malware

If malware was found on a computer, it must be removed.

However, simply deleting one malicious file may not be enough.

For example:

```text
Malware File
     ↓
Persistence Mechanism
     ↓
Malware Starts Again
```

Therefore, investigators should also look for other components that allow the malware to return.

> 🔍 **Investigation:** Look for persistence mechanisms, additional malicious files, compromised accounts, scheduled tasks, registry changes, and other signs of attacker activity.

---

## 💻 Rebuild Compromised Systems

In some cases, rebuilding a system is safer than trying to clean it manually.

For example:

```text
Compromised System
       ↓
Backup Important Data
       ↓
Reinstall Operating System
       ↓
Apply Security Updates
       ↓
Restore Clean Data
       ↓
Verify Security
```

This can help remove hidden malware or attacker changes that may be difficult to find.

---

## 🛡️ System Hardening

During eradication, the security team may also improve the security of systems.

This is called **system hardening**.

Examples include:

* Disabling unnecessary services
* Removing unnecessary accounts
* Applying security patches
* Improving firewall rules
* Enabling stronger authentication
* Improving endpoint security
* Limiting user permissions

> 💡 **Key idea:** Hardening should not only be performed on the infected computer. In some incidents, similar weaknesses may exist across many systems.

---

# ♻️ 3. Recovery

**Recovery** means bringing the affected systems back to **normal business operation**.

Before returning a system to production, the security team should verify that it is:

* Working correctly
* Secure
* Properly configured
* Connected to the correct systems
* Containing the required business data

The basic process can look like this:

```text
Eradication Complete
        ↓
Restore / Rebuild System
        ↓
Security Verification
        ↓
Business Verification
        ↓
Return to Production
```

---

## 🧪 Verify the System

Before returning a system to production, both technical and business teams may need to verify it.

### Security Team

Checks things such as:

* No known malware remains
* Security patches are installed
* Security tools are working
* Logs are being collected
* Monitoring is enabled

### Business Team

Checks things such as:

* Applications work correctly
* Required data is available
* Business processes work normally

> 📌 **Important:** A system should not be returned to normal operation simply because it starts successfully. It should also be checked for security and business functionality.

---

# 👀 Increased Monitoring After Recovery

Recovered systems should receive **strong monitoring** after an incident.

Why?

Because a compromised system may become a target again if the attacker can regain access.

Security teams should watch for unusual activity such as:

### 👤 Unusual Logins

For example:

```text
Normal:
User A → Computer A

Suspicious:
User A → Computer Z
```

A user or service account logging into a system where it has never logged in before may require investigation.

---

### ⚙️ Unusual Processes

Look for processes that:

* Have never appeared on the system before
* Run from unusual locations
* Use suspicious command-line arguments
* Start unexpectedly

Example:

```text
Normal:
C:\Windows\System32\svchost.exe

Suspicious:
C:\Users\Public\svchost.exe
```

> ⚠️ **Note:** An unusual process is not automatically malicious. It is a signal that should be investigated.

---

### 📝 Unusual Registry Changes

Windows malware may modify specific registry locations to create persistence.

For example:

```text
Registry
   ↓
Startup / Run Key
   ↓
Malware Starts Automatically
```

Therefore, unexpected registry changes should be investigated.

---

# 📅 Recovery Can Take Months

For a large security incident, recovery may not happen in one day.

Organizations may recover in several phases.

```text
Phase 1
Quick Security Improvements
        ↓
Phase 2
Remove Remaining Weaknesses
        ↓
Phase 3
Long-Term Security Improvements
        ↓
Phase 4
Stable and Secure Environment
```

---

## 🟢 Early Recovery Phase

The first phase usually focuses on **quick security improvements**.

Examples:

* Fixing easy-to-exploit weaknesses
* Applying important patches
* Disabling unnecessary services
* Resetting compromised credentials
* Improving monitoring

These actions can provide quick security improvements.

---

## 🔵 Later Recovery Phases

Later phases focus on **long-term improvements**.

Examples include:

* Improving network architecture
* Improving identity and access management
* Deploying better security monitoring
* Improving incident-response procedures
* Replacing outdated systems
* Improving security policies and controls

> 💡 **Key idea:** Recovery is not only about getting systems working again. It is also an opportunity to make the environment stronger than it was before the incident.

---

# 🔄 Complete Process

The three stages work together:

```text
┌──────────────────┐
│   Containment    │
│                  │
│ Stop the spread  │
│ and limit damage │
└────────┬─────────┘
         ↓
┌──────────────────┐
│   Eradication    │
│                  │
│ Remove malware   │
│ and root cause   │
└────────┬─────────┘
         ↓
┌──────────────────┐
│     Recovery     │
│                  │
│ Restore systems  │
│ and monitor them │
└──────────────────┘
```

### 🧠 In Simple Words

Think of a cybersecurity incident like a fire:

```text
🔥 Incident
    ↓
🛑 Containment
Stop the fire from spreading
    ↓
🧹 Eradication
Remove the source of the fire
    ↓
🏠 Recovery
Repair the building and return to normal
    ↓
👀 Monitoring
Watch for signs of another fire
```

The comparison is not exact, but it helps explain the basic idea.

---

# ✅ Key Takeaways

1. **Containment** stops the incident from spreading and causing more damage.
2. **Short-term containment** focuses on quick actions with minimal changes to the affected systems.
3. **Long-term containment** uses more permanent changes such as password resets, firewall rules, patches, and security controls.
4. **Eradication** removes malware, attacker access, and the root cause of the incident.
5. **Recovery** returns systems to normal business operation.
6. Always consider **evidence preservation** before shutting down an affected system.
7. Recovered systems should receive **extra monitoring** after an incident.
8. Watch for unusual **logins, processes, and registry changes**.
9. Large incidents may require **months of recovery work**.
10. Recovery is also an opportunity to **improve the organization's security** and reduce the chance of another incident.
