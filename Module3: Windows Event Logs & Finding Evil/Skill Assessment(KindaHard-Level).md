# 🛡️ SOC Investigation — Windows Log Analysis Cheat Sheet

> **Module 3 | Threat Hunting via Windows Event Logs**

---

## 📌 Core Concept — Event IDs You Must Know

| Sysmon Event ID | What It Logs       | Used For                                      |
| --------------- | ------------------ | --------------------------------------------- |
| 1               | Process creation   | PPID spoofing, suspicious child processes     |
| 3               | Network connection | C2 communication, unexpected outbound traffic |
| 7               | Image/DLL loaded   | DLL hijacking, CLR injection                  |
| 8               | CreateRemoteThread | Process injection                             |
| 10              | ProcessAccess      | LSASS dumping, credential theft               |
| 11              | FileCreate         | Dump files created on disk                    |

| Security Event ID | What It Logs     | Used For                   |
| ----------------- | ---------------- | -------------------------- |
| 4624              | Successful logon | Post-attack credential use |
| 4625              | Failed logon     | Brute force detection      |
| 4688              | Process creation | Parent-child relationships |
| 4663              | File access      | Sensitive file access      |

---

# 🔍 Investigation Framework — 6 Question Types

---

## Q1 — DLL Hijacking

**Question pattern:** "Find the process responsible for DLL hijacking"

**Event ID:** Sysmon 7 (ImageLoad)

### 🔎 What to look for:

* DLL loaded from wrong/unexpected path
* `Signed: false`
* Known DLL name but wrong location
* Process running from user-writable folder (Downloads, ProgramData, AppData)

### 📌 Key fields:

```text
Image       = process that loaded the DLL (your answer)
ImageLoaded = path of the DLL loaded
Signed      = false (red flag)
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -FilterHashtable @{
    Path = $Path
    Id   = 7
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    $dll    = ($data | Where-Object Name -eq 'ImageLoaded').'#text'
    $signed = ($data | Where-Object Name -eq 'Signed').'#text'

    if ($signed -eq 'false' -or
       ($dll -notlike '*\Windows\*' -and
        $dll -notlike '*\Program Files\*')) {
        [PSCustomObject]@{
            Time      = $_.TimeCreated
            Process   = ($data | Where-Object Name -eq 'Image').'#text'
            DLLLoaded = $dll
            Signed    = $signed
        }
    }
} | Format-List
```

### 🚨 Red flags:

```text
✅ Process in C:\ProgramData\ or C:\Users\*\Downloads\
✅ DLL unsigned (Signed: false)
✅ Known Windows DLL name loaded from wrong folder
```

### 💡 Real example:

```text
Process:   C:\ProgramData\Dism.exe        ← wrong location
DLL:       C:\Windows\System32\urlmon.dll ← internet capability
Signed:    false
→ Answer: Dism.exe
```
<img width="609" height="222" alt="image" src="https://github.com/user-attachments/assets/9d61d021-0e92-4898-943a-bb3985d92a00" />

---

## Q2 — Unmanaged PowerShell (CLR Injection)

**Question pattern:** "Find the process that executed unmanaged PowerShell code"

**Event ID:** Sysmon 7 (ImageLoad)

### 🔎 What to look for:

* `clr.dll` OR `clrjit.dll` loaded inside a NON-managed process
* Any process that is not powershell.exe, msbuild.exe, or known .NET app

### 📌 Key fields:

```text
Image       = process that loaded CLR (your answer)
ImageLoaded = clr.dll or clrjit.dll (the signal)
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -Path $Path |
 Where-Object { $_.Id -eq 7 } |
 ForEach-Object {
     $xml = [xml]$_.ToXml()
     $data = $xml.Event.EventData.Data
     [PSCustomObject]@{
         Time            = $_.TimeCreated
         Process         = ($data | Where-Object {$_.Name -eq 'Image'}).'#text'
         DLLLoaded       = ($data | Where-Object {$_.Name -eq 'ImageLoaded'}).'#text'
         Signed          = ($data | Where-Object {$_.Name -eq 'Signed'}).'#text'
         SignatureStatus = ($data | Where-Object {$_.Name -eq 'SignatureStatus'}).'#text'
     }
 } | Format-Table -AutoSize
```

### 🚨 Red flags:

<img width="1832" height="312" alt="image" src="https://github.com/user-attachments/assets/a9bbb257-29f9-47a6-ade5-4ca581c548b7" />

<img width="2105" height="513" alt="image" src="https://github.com/user-attachments/assets/4fd9baa4-0811-431f-915c-13a46d6334b4" />

---

## Q3 — Process Injection (Who Performed It)

**Question pattern:** "Find the process that injected into [victim process]"

**Event ID:** Sysmon 8 (CreateRemoteThread)

### 🔎 What to look for:

* `SourceImage = injector (the answer)`
* `TargetImage = victim process found in Q2`

### 📌 Key fields:

```text
SourceImage = process that PERFORMED injection (your answer)
TargetImage = process that got injected INTO (Q2 answer)
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -FilterHashtable @{
    Path = $Path
    Id   = 8
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    [PSCustomObject]@{
        Time     = $_.TimeCreated
        Injector = ($data | Where-Object Name -eq 'SourceImage').'#text'
        Victim   = ($data | Where-Object Name -eq 'TargetImage').'#text'
    }
} | Format-List
```

### 🚨 Red flags:

```text
✅ Injector is a LOLBin (rundll32, mshta, regsvr32)
✅ Victim matches process from Q2
✅ Injector has no business touching victim process
```

### 💡 Real example:

```text
Injector: C:\Windows\System32\rundll32.exe
Victim:   C:\Program Files\WindowsApps\...Calculator.exe
→ Answer: rundll32.exe
```
<img width="1487" height="277" alt="image" src="https://github.com/user-attachments/assets/6bf6724b-76aa-4c9c-959f-263ba916ed4f" />

---

## Q4 — LSASS Dump (Credential Theft)

**Question pattern:** "Find the process that performed an LSASS dump"

**Event ID:** Sysmon 10 (ProcessAccess)

### 🔎 What to look for:

* `TargetImage = lsass.exe`
* `GrantedAccess = high privilege value`

### 📌 Key fields:

```text
SourceImage   = process performing the dump (your answer)
TargetImage   = lsass.exe (the target)
GrantedAccess = access rights requested
```

### ⚠️ Common suspicious GrantedAccess values:

```text
0x1fffff = PROCESS_ALL_ACCESS (full access — very suspicious)
0x1F0FFF = Full access
0x1010   = Read + query (common Mimikatz value)
0x143A   = Common in other dump tools
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -FilterHashtable @{
    Path = 'C:\Logs\Dump\*.evtx'
    Id   = 10
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    [PSCustomObject]@{
        Time          = $_.TimeCreated
        Dumper        = ($data | Where-Object Name -eq 'SourceImage').'#text'
        Target        = ($data | Where-Object Name -eq 'TargetImage').'#text'
        GrantedAccess = ($data | Where-Object Name -eq 'GrantedAccess').'#text'
    }
} |
Where-Object { $_.Target -like '*lsass*' } |
Format-List
```

### 🚨 Red flags:

```text
✅ Any process opening lsass.exe with high access rights
✅ Tool running from Downloads or temp folder
✅ GrantedAccess 0x1fffff or 0x1F0FFF
✅ Legitimate-looking tool used maliciously (ProcessHacker, ProcDump)
```

### 💡 Real example:

```text
Dumper:        C:\Users\waldo\Downloads\...\ProcessHacker.exe
Target:        C:\Windows\System32\lsass.exe
GrantedAccess: 0x1fffff
→ Answer: ProcessHacker.exe
```
<img width="1163" height="972" alt="image" src="https://github.com/user-attachments/assets/80c3187f-e5f0-480c-a43b-a47837bdc2a8" />

---

## Q5 — Post-Dump Logon (Were Credentials Used?)

**Question pattern:** "Did an ill-intended login take place after the LSASS dump?"

**Event ID:** Security 4624 (Successful Logon)

### 🔎 What to look for:

* Successful logon AFTER dump timestamp
* Logon Type 3 (Network/lateral movement)
* Unexpected account or source IP

### 📌 Key fields:

```text
Time        = must be AFTER dump timestamp
Account     = who logged in
LogonType   = 3 (network) is most suspicious
SourceIP    = where logon came from
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -FilterHashtable @{
    Path = $Path
    Id   = 4624
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    [PSCustomObject]@{
        Time           = $_.TimeCreated
        Account        = ($data | Where-Object Name -eq 'TargetUserName').'#text'
        Domain         = ($data | Where-Object Name -eq 'TargetDomainName').'#text'
        LogonType      = ($data | Where-Object Name -eq 'LogonType').'#text'
        SourceIP       = ($data | Where-Object Name -eq 'IpAddress').'#text'
        WorkStation    = ($data | Where-Object Name -eq 'WorkstationName').'#text'
        Authentication = ($data | Where-Object Name -eq 'AuthenticationPackageName').'#text'
    }
} | Format-List
```

### ⚠️ Important note:

```text
No events found ≠ credentials definitely not stolen
It means: no malicious logon IN THESE LOGS

Attacker may have:
→ Used credentials on different machine
→ Not used them yet
→ Failed to extract useful credentials
```

### 💡 Real example:

```text
No 4624 events found after dump timestamp
→ Answer: No
```
<img width="1069" height="148" alt="image" src="https://github.com/user-attachments/assets/4ca51259-f465-4730-a749-877a7b48c7ab" />

---

## Q6 — PPID Spoofing (Strange Parent-Child)

**Question pattern:** "Find the process temporarily executed via strange parent-child relationship"

**Event ID:** Sysmon 1 (Process Creation)

### 🔎 What to look for:

* Parent process that has no business spawning child
* Child is a shell or recon tool (`cmd.exe`, `powershell.exe`, `whoami.exe`)
* Attacker checking privileges (`whoami`, `ipconfig`, `net user`)

### 📌 Key fields:

```text
Child       = process that was spawned (your answer)
Parent      = process that spawned it (the spoofed parent)
CommandLine = what command was run
```

### 🛠️ PowerShell Command:

```powershell
Get-WinEvent -FilterHashtable @{
    Path = $Path
    Id   = 1
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    [PSCustomObject]@{
        Time        = $_.TimeCreated
        Child       = ($data | Where-Object Name -eq 'Image').'#text'
        Parent      = ($data | Where-Object Name -eq 'ParentImage').'#text'
        CommandLine = ($data | Where-Object Name -eq 'CommandLine').'#text'
    }
} | Format-List
```

### 🚨 Red flags:

```text
✅ WerFault.exe → cmd.exe (error reporter spawning shell)
✅ winword.exe  → powershell.exe (Office spawning PowerShell)
✅ notepad.exe  → whoami.exe (text editor running recon)
✅ Any non-shell process spawning cmd/powershell
✅ whoami, ipconfig, net user in CommandLine = recon activity
```

### 💡 Real example:

```text
Parent:      C:\Windows\System32\WerFault.exe
Child:       C:\Windows\System32\cmd.exe
CommandLine: cmd.exe /c whoami
→ Answer: cmd.exe
```
<img width="735" height="452" alt="image" src="https://github.com/user-attachments/assets/a6d3f8be-ad1e-48b9-8656-13f9ece84efc" />

---

## 📌 Quick Reference — Which Event ID For Which Question

```text
Suspicious DLL loaded?          → Sysmon ID 7
CLR in wrong process?           → Sysmon ID 7
Who performed injection?        → Sysmon ID 8
LSASS accessed?                 → Sysmon ID 10
Successful logon after attack?  → Security ID 4624
Strange parent-child process?   → Sysmon ID 1
```

---

## 🛠️ PowerShell Template — Reuse For Any Question

```powershell
# Change Path and Id to match your question
Get-WinEvent -FilterHashtable @{
    Path = $Path
    Id   = <EventID>
} |
ForEach-Object {
    $xml  = [xml]$_.ToXml()
    $data = $xml.Event.EventData.Data
    [PSCustomObject]@{
        Time   = $_.TimeCreated
        Field1 = ($data | Where-Object Name -eq '<FieldName>').'#text'
        Field2 = ($data | Where-Object Name -eq '<FieldName>').'#text'
    }
} | Format-List
```

---

## 🧠 Investigation Mindset

```text
1. Read the question carefully
   → What technique is being described?

2. Pick the right Event ID
   → Use the quick reference above

3. Run the command
   → Use Format-List to avoid truncation

4. Look for anomalies
   → Wrong path, wrong parent, wrong DLLs, wrong access rights

5. Confirm with context
   → Does this make sense given what we know?
   → Cross-reference timestamps across questions
```

---

## 🗺️ MITRE ATT&CK Mapping

| Technique            | ID        | Event ID      | Key Signal                  |
| -------------------- | --------- | ------------- | --------------------------- |
| DLL Hijacking        | T1574.001 | Sysmon 7      | Wrong DLL path              |
| Unmanaged PowerShell | T1059.001 | Sysmon 7      | clr.dll in wrong process    |
| Process Injection    | T1055     | Sysmon 8      | CreateRemoteThread          |
| LSASS Dumping        | T1003.001 | Sysmon 10     | High GrantedAccess on lsass |
| Pass-the-Hash        | T1550.002 | Security 4624 | NTLM logon after dump       |
| PPID Spoofing        | T1134.004 | Sysmon 1      | Strange parent-child        |
