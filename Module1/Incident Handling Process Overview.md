## Incident Handling Process

The Incident Handling Process is the structured way organizations prepare for, detect, and respond to security incidents. Defined by NIST, it consists of 4 main stages.

> 🚨 **Important:** This is not a 1-to-1 match with the Kill Chain stages. They are separate frameworks used together.

<img width="1382" height="657" alt="image" src="https://github.com/user-attachments/assets/0d428b36-95c1-4371-b4b2-74b90b958d70" />

---

### The 4 Stages (NIST)
**Preparation ➔ Detection & Analysis ➔ Containment, Eradication & Recovery ➔ Post-Incident Activity**

It is cyclic, not linear, new evidence discovered mid-investigation can send you back to earlier steps.

---

### Where You Spend Most of Your Time

The majority of an incident handler's work lives in the first two stages:

* **Preparation:** Building capabilities, tools, and playbooks before anything happens.
* **Detection & Analysis:** Continuously monitoring and investigating suspicious events.

Even when an active incident is being handled, resources must keep running in these two stages. You cannot pause detection just because you are responding to an active incident.

---

### The Two Core Activities Inside the Process

#### 1. Investigation

* Find patient zero, the first infected machine or user.
* Build a timeline of what happened (and what is still happening if the attack is ongoing).
* Identify the tools and malware the attacker used.
* Document every compromised system and every attacker action.

#### 2. Recovery

* Create and implement a recovery plan.
* Restore normal business operations.

> 🎯 **Critical Rule: Don't Skip or Half-Complete Steps**  
> If you find 10 infected machines, contain and eradicate all 10 before moving on.  
>  
> **Containing only 5 while leaving 5 infected means:**  
> * You are alerting the attacker that you have found them.  
> * They can change behavior, cover tracks, or escalate damage.  
> * Your response becomes unpredictable and potentially ineffective.  
>  
> Complete each step fully before moving to the next.

---

### After the Incident

Once fully handled:

* A report is written covering the cause and cost of the incident.
* Lessons learned activities are conducted, focusing on what should change to prevent this from happening again.

---

### Quick Summary

| Activity | Purpose |
| :--- | :--- |
| **Preparation** | Be ready before incidents happen |
| **Detection & Analysis** | Find and understand the incident |
| **Containment, Eradication & Recovery** | Stop it, remove it, restore operations |
| **Post-Incident** | Report + improve |
