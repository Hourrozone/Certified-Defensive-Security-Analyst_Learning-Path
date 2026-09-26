# Skills Assessment

# Hunting For Stuxbot (Round 2)

> 🧠 **Recently uncovered details shed light on the operational strategy of Stuxbot's newest iteration.**

🚨 **The newest iterations of Stuxbot are exploiting the `C:\Users\Public` directory as a conduit for deploying supplementary utilities.**

🚨 **The newest iterations of Stuxbot are utilizing registry run keys as a mechanism to ensure their sustained presence within the infected system.**

🚨 **The newest iterations of Stuxbot are utilizing PowerShell Remoting for lateral movement within the network and to gain access to domain controllers.**

---

# The Available Data

🖥️ **The cybersecurity strategy implemented is predicated on the utilization of the Elastic stack as a SIEM solution. Through the "Discover" functionality we can see logs from multiple sources. These sources include:**

* 🖥️ Windows audit logs (categorized under the index pattern `windows*`)
* 🔎 System Monitor (Sysmon) logs (also falling under the index pattern `windows*`, more about Sysmon here)
* ⚡ PowerShell logs (indexed under `windows*` as well, more about PowerShell logs here)
* 🌐 Zeek logs, a network security monitoring tool (classified under the index pattern `zeek*`)

---

# The Tasks

🎯 **Navigate to the bottom of this section and click on Click here to spawn the target system!**

🖥️ **click on the side navigation toggle, and click on "Discover". Then, click on the calendar icon, specify "last 15 years", and click on "Apply".**

---

## 🔎 Hunt 1: Create a KQL query to hunt for "Lateral Tool Transfer" to C:\Users\Public.

📌 **Enter the content of the user.name field in the document that is related to a transferred tool that starts with "r" as your answer.**

---

## 🔎 Hunt 2: Create a KQL query to hunt for "Boot or Logon Autostart Execution: Registry Run Keys / Startup Folder".

📌 **Enter the content of the registry.value field in the document that is related to the first registry-based persistence action as your answer.**

---

## 🔎 Hunt 3: Create a KQL query to hunt for "PowerShell Remoting for Lateral Movement".

📌 **Enter the content of the winlog.user.name field in the document that is related to PowerShell remoting-based lateral movement towards DC1.**

---

# 🧠 General Hunting Process

1. 📖 Read the technique name → go to **MITRE ATT&CK** and the provided reference link
2. 🔎 Identify **what artifact** the technique leaves behind (file, registry, log)
3. 🧩 Map it to the **correct Event ID**
4. 🧪 Build a **KQL query** in Elastic
5. 🔍 Filter results and look for the suspicious document
6. 📌 Extract the answer field from that document

---

# 🔴 Hunt 1 — Lateral Tool Transfer (T1570)

🎯 **Goal:** Find a tool transferred to `C:\Users\Public`

### 💡 Why `C:\Users\Public`?

It's a world-writable folder — any user can write to it, making it a common attacker staging ground.

### 🖥️ Event ID:

**Event ID:** `11` (Sysmon — File Created)

### 🔎 KQL:

```kql
event.code: "11" AND file.path: *C:\Users\Public*
```

### 📌 Answer field:

`user.name`

### ✅ Answer:

`svc-sql1`

<img width="2156" height="1137" alt="image_2026-09-27_00-31-03" src="https://github.com/user-attachments/assets/f9a1016e-4a81-4d41-b13e-86facbb23aae" />

---

# 🔴 Hunt 2 — Registry Run Key Persistence (T1547.001)

🎯 **Goal:** Find attacker-created registry run key for persistence

### 💡 Why Run Keys?

Anything written to these keys **executes automatically on startup** — attackers use this to survive reboots.

### 📚 Reference (MITRE T1547.001): Default Run Keys on Windows:

* `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce`
* `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run`
* `HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce`

### 🖥️ Event ID:

**Event ID:** `13` (Sysmon — Registry Value Set)

### 🔎 KQL:

```kql
event.code: "13" AND registry.path: (*CurrentVersion\\Run* OR *CurrentVersion\\RunOnce*)
```

> ⚠️ **Sort results by `@timestamp` ascending to find the first persistence action**

### 📌 Answer field:

`registry.value`

### ✅ Answer:

`LgvHsviAUVTsIN`

<img width="2150" height="1072" alt="image_2026-09-27_00-31-03 (2)" src="https://github.com/user-attachments/assets/2fb137e3-4ffb-40bb-a7a6-951b5ddf32b5" />

---

# 🔴 Hunt 3 — PowerShell Remoting Lateral Movement (T1021.006)

🎯 **Goal:** Find PS remoting activity targeting DC1

### 📚 Reference:

[WinRM for Lateral Movement](https://www.ired.team/offensive-security/lateral-movement/t1028-winrm-for-lateral-movement)

### 💡 How PS Remoting works (2 steps):

```powershell
# Step 1 — Attacker creates remote session to target
New-PSSession -ComputerName DC1 -Credential (Get-Credential)

# Step 2 — Attacker enters that session
Enter-PSSession 1
# Now running commands directly on DC1
```

> ⚠️ **Key lesson:** These are two separate actions — DC1's hostname only appears in `New-PSSession`, NOT in `Enter-PSSession`. Always hunt for both commands together.

### 🖥️ Event ID:

**Event ID:** `4104` (PowerShell Script Block Logging)

### 🔎 KQL:

```kql
event.code: "4104" AND powershell.file.script_block_text: (*New-PSSession* OR *Enter-PSSession*)
```

### 📌 Answer field:

`winlog.user.name`

### ✅ Answer:

`svc-sql1`

---

# 📌 Quick Reference — Event IDs Used

| Event ID | Source     | What It Captures              |
| -------- | ---------- | ----------------------------- |
| `11`     | Sysmon     | File created on disk          |
| `13`     | Sysmon     | Registry value set            |
| `4104`   | PowerShell | Script block content executed |

<img width="2141" height="1143" alt="image_2026-09-27_00-31-03 (3)" src="https://github.com/user-attachments/assets/6920dbc1-0992-405a-b907-3fd3879889a2" />

---

# 💡 Tips for Future Hunts

* 📚 Always **check MITRE** and the provided reference link first — they tell you exactly what artifacts to look for
* 🚩 **Random-looking strings** in registry keys, process names, or file names = red flag
* ⚠️ PowerShell remoting is a **two-step action** — hunt `New-PSSession` and `Enter-PSSession` together
* 🔎 When looking for the **first** occurrence of something, always sort by `@timestamp` ascending
* 🚨 `C:\Users\Public` appearing in file paths = common attacker staging location
