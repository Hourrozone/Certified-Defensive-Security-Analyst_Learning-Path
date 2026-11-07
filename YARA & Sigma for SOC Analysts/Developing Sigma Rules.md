<img width="633" height="113" alt="image" src="https://github.com/user-attachments/assets/32eb6a0a-82ed-4da6-b482-6b5e7b9fd6ec" /><img width="745" height="57" alt="image" src="https://github.com/user-attachments/assets/2689837d-faf9-4890-a774-a92fa12b59a4" /># Developing Sigma Rules

In this section, we'll cover manual Sigma rule development.

## Manually Developing a Sigma Rule

### Example 1: LSASS Credential Dumping

Let's dive into the world of Sigma rules using a sample named `shell.exe` (a renamed version of mimikatz) residing in the `C:\Samples\YARASigma` directory of this section's target as an illustration. We want to understand the process behind crafting a Sigma rule, so let's get our hands dirty.

After executing `shell.exe` as follows, we collected the most critical events and saved them as `lab_events.evtx` inside the `C:\Events\YARASigma` directory of this section's target.

The process created by `shell.exe` (mimikatz) will try to access the process memory of `lsass.exe`. The system monitoring tool Sysmon was running in the background and captured this activity in the event logs (Event ID 10).

<img width="717" height="835" alt="image" src="https://github.com/user-attachments/assets/10700f85-400c-4ea6-869d-d4ca1608d3df" />

First off, Sysmon Event ID 10 is triggered when a process accesses another process, and it logs the permission flags in the `GrantedAccess` field. This event log contains two important fields, `TargetImage` and `GrantedAccess`. In a typical LSASS memory dumping scenario, the malicious process needs specific permissions to access the memory space of the LSASS process. These permissions are often read/write access, among other things.

<img width="1063" height="606" alt="image" src="https://github.com/user-attachments/assets/fe9a13dd-04b8-4339-8997-2227315607b4" />

Now, why is `0x1010` crucial here? This hexadecimal flag essentially combines `PROCESS_VM_READ (0x0010)` and `PROCESS_QUERY_INFORMATION (0x0400)` permissions. To translate that: the process is asking for read access to the virtual memory of LSASS and the ability to query certain information from the process. While `0x0410` is the most common GrantedAccess flag used for reading LSASS memory, `0x1010` implies both reading and querying information from the process and is also frequently observed during credential dumping attacks.

So how can we weaponize this information for detection? Well, in our security monitoring stack, we would configure Sysmon to flag or alert on any Event ID 10 where the `TargetImage` is `lsass.exe` and `GrantedAccess` is set to `0x1010`.

A Sigma rule that checks for the abovementioned conditions can be found below.

<img width="741" height="385" alt="image" src="https://github.com/user-attachments/assets/fe46b2e3-6597-4391-ae5e-b42d8fd6b8ad" />

#### Sigma Rule Breakdown

- **`title`:** This title offers a concise overview of the rule's objective, specifically aimed at detecting interactions with LSASS memory involving a particular access flag.

<img width="725" height="55" alt="image" src="https://github.com/user-attachments/assets/661a55fc-057f-4834-9815-5c696526ff95" />

- **`status`:** This field signals that the rule is in the testing phase, suggesting that additional fine-tuning or validation may be necessary.

<img width="745" height="57" alt="image" src="https://github.com/user-attachments/assets/00b97359-4da0-4c10-8d3d-d72ebcc14920" />

- **`description`:** Rule description.

<img width="752" height="69" alt="image" src="https://github.com/user-attachments/assets/5256f7af-8c62-4104-90b6-f99703d1f8ba" />

- **`date`:** This field marks the date when the rule was either updated or originally created.

<img width="754" height="62" alt="image" src="https://github.com/user-attachments/assets/d286c2ed-bb52-46cd-9da6-b6289ab78c24" />

- **`tags`:** The rule is tagged with `attack.credential_access` and `attack.t1003.001`. These tags help categorize the rule based on known attack techniques or tactics related to credential access.

<img width="751" height="103" alt="image" src="https://github.com/user-attachments/assets/34438ce9-9207-45d6-adcc-a16f6e217852" />

- **`logsource`:** The logsource specifies the log source that the rule is intended to analyze. It contains `category` as `process_access` which indicates that the rule focuses on log events related to process access (Sysmon Event ID 10, if we use Sigma's default config files). Also, `product: windows` specifies that the rule is specifically designed for Windows operating systems.

<img width="735" height="104" alt="image" src="https://github.com/user-attachments/assets/39865857-fa14-472d-aed7-680af477f113" />

- **`detection`:** The detection section defines the conditions that must be met for the rule to trigger an alert. The selection part specifies the criteria for selecting relevant log events where the `TargetImage` field ends with `\lsass.exe` and `GrantedAccess` field ends with the hexadecimal value `0x1010`. The `GrantedAccess` field represents the access rights or permissions associated with the process. In this case, it targets events with a specific access flag of `0x1010`. Finally, the condition part specifies that the selection criteria must be met for the rule to trigger an alert. In this case, both the `TargetImage` and `GrantedAccess` criteria must be met.

<img width="748" height="151" alt="image" src="https://github.com/user-attachments/assets/f1242f3c-88f5-4605-a79b-657017798728" />

Οur first Sigma rule above can be found inside the `C:\Rules\sigma` directory of this section's target as `proc_access_win_lsass_access.yml`. Let's explore the `sigmac` tool that can help us transform this rule into queries or configurations compatible with a multitude of SIEMs, log management solutions, and other security analytics tools.

The `sigmac` tool can be found inside the `C:\Tools\sigma-0.21\tools` directory of this section's target.

Suppose that we wanted to convert our Sigma rule into a PowerShell (Get-WinEvent) query. This could have been accomplished with the help of `sigmac` as follows.

<img width="737" height="124" alt="image" src="https://github.com/user-attachments/assets/09b1e4c3-ed95-4b9b-b019-2e779e5089de" />

Let's adjust the Get-WinEvent query above by specifying the `.evtx` file that is related to LSASS access by another process (`lab_events.evtx` inside the `C:\Events\YARASigma` directory of this section's target) and see if it will identify the Sysmon event (ID 10) that we analyzed at the beginning of this section.

**Note:** Please open a PowerShell terminal as administrator to run the query.

<img width="738" height="667" alt="image" src="https://github.com/user-attachments/assets/ab71bbbb-4f43-4498-892a-f21e766af237" />

The related Sysmon event with ID 10 is successfully identified!

But let's not stop there - remember, false positives are the enemy of effective security monitoring.

We should also cross-reference the `SourceImage` (the process initiating the access) against a list of known, safe processes that commonly interact with LSASS.

If we see an unfamiliar or unusual process trying to read LSASS with a `GrantedAccess` that ends with `10`, `30`, `50`, `70`, `90`, `B0`, `D0`, `F0`, `18`, `38`, `58`, `78`, `98`, `B8`, `D8`, `F8`, `1A`, `3A`, `5A`, `7A`, `9A`, `BA`, `DA`, `FA`, `0x14C2`, and `FF` (these suffixes come from studying the GrantedAccess values that various LSASS credential dumping techniques require), that's a red flag, and our incident response protocol should kick in.

Especially, if the `SourceImage` resides in suspicious paths containing, `\Temp\`, `\Users\Public\`, `\PerfLogs\`, `\AppData\`, `\htb\` etc. that's another red flag, and our incident response protocol should kick in.

A more robust version of the Sigma rule we created taking the above points into consideration can be found inside the `C:\Rules\sigma` directory of this section's target as `proc_access_win_lsass_access_robust.yml`

```yaml
title: LSASS Access From Program in Potentially Suspicious Folder
id: fa34b441-961a-42fa-a100-ecc28c886725
status: experimental
description: Detects process access to LSASS memory with suspicious access flags and from a potentially suspicious folder
references:
    - https://docs.microsoft.com/en-us/windows/win32/procthread/process-security-and-access-rights
    - https://onedrive.live.com/view.aspx?resid=D026B4699190F1E6!2843&ithint=file%2cpptx&app=PowerPoint&authkey=!AMvCRTKB_V1J5ow
    - https://web.archive.org/web/20230208123920/https://cyberwardog.blogspot.com/2017/03/chronicles-of-threat-hunter-hunting-for_22.html
    - https://www.slideshare.net/heirhabarov/hunting-for-credentials-dumping-in-windows-environment
    - http://security-research.dyndns.org/pub/slides/FIRST2017/FIRST-2017_Tom-Ueltschi_Sysmon_FINAL_notes.pdf
author: Florian Roth (Nextron Systems)
date: 2021/11/27
modified: 2023/05/05
tags:
    - attack.credential_access
    - attack.t1003.001
    - attack.s0002
logsource:
    category: process_access
    product: windows
detection:
    selection:
        TargetImage|endswith: '\lsass.exe'
        GrantedAccess|endswith:
            - '10'
            - '30'
            - '50'
            - '70'
            - '90'
            - 'B0'
            - 'D0'
            - 'F0'
            - '18'
            - '38'
            - '58'
            - '78'
            - '98'
            - 'B8'
            - 'D8'
            - 'F8'
            - '1A'
            - '3A'
            - '5A'
            - '7A'
            - '9A'
            - 'BA'
            - 'DA'
            - 'FA'
            - '0x14C2'  # https://github.com/b4rtik/ATPMiniDump/blob/76304f93b390af3bb66e4f451ca16562a479bdc9/ATPMiniDump/ATPMiniDump.c
            - 'FF'
        SourceImage|contains:
            - '\Temp\'
            - '\Users\Public\'
            - '\PerfLogs\'
            - '\AppData\'
            - '\htb\'
    filter_optional_generic_appdata:
        SourceImage|startswith: 'C:\Users\'
        SourceImage|contains: '\AppData\Local\'
        SourceImage|endswith:
            - '\Microsoft VS Code\Code.exe'
            - '\software_reporter_tool.exe'
            - '\DropboxUpdate.exe'
            - '\MBAMInstallerService.exe'
            - '\WebexMTA.exe'
            - '\WebEx\WebexHost.exe'
            - '\JetBrains\Toolbox\bin\jetbrains-toolbox.exe'
        GrantedAccess: '0x410'
    filter_optional_dropbox_1:
        SourceImage|startswith: 'C:\Windows\Temp\'
        SourceImage|endswith: '.tmp\DropboxUpdate.exe'
        GrantedAccess:
            - '0x410'
            - '0x1410'
    filter_optional_dropbox_2:
        SourceImage|startswith: 'C:\Users\'
        SourceImage|contains: '\AppData\Local\Temp\'
        SourceImage|endswith: '.tmp\DropboxUpdate.exe'
        GrantedAccess: '0x1410'
    filter_optional_dropbox_3:
        SourceImage|startswith:
            - 'C:\Program Files (x86)\Dropbox\'
            - 'C:\Program Files\Dropbox\'
        SourceImage|endswith: '\DropboxUpdate.exe'
        GrantedAccess: '0x1410'
    filter_optional_nextron:
        SourceImage|startswith:
            - 'C:\Windows\Temp\asgard2-agent\'
            - 'C:\Windows\Temp\asgard2-agent-sc\'
        SourceImage|endswith:
            - '\thor64.exe'
            - '\thor.exe'
            - '\aurora-agent-64.exe'
            - '\aurora-agent.exe'
        GrantedAccess:
            - '0x1fffff'
            - '0x1010'
            - '0x101010'
    filter_optional_ms_products:
        SourceImage|startswith: 'C:\Users\'
        SourceImage|contains|all:
            - '\AppData\Local\Temp\'
            - '\vs_bootstrapper_'
        GrantedAccess: '0x1410'
    filter_optional_chrome_update:
        SourceImage|startswith: 'C:\Program Files (x86)\Google\Temp\'
        SourceImage|endswith: '.tmp\GoogleUpdate.exe'
        GrantedAccess:
            - '0x410'
            - '0x1410'
    filter_optional_keybase:
        SourceImage|startswith: 'C:\Users\'
        SourceImage|endswith: \AppData\Local\Keybase\keybase.exe
        GrantedAccess: '0x1fffff'
    filter_optional_avira:
        SourceImage|contains: '\AppData\Local\Temp\is-'
        SourceImage|endswith: '.tmp\avira_system_speedup.tmp'
        GrantedAccess: '0x1410'
    filter_optional_viberpc_updater:
        SourceImage|startswith: 'C:\Users\'
        SourceImage|contains: '\AppData\Roaming\ViberPC\'
        SourceImage|endswith: '\updater.exe'
        TargetImage|endswith: '\winlogon.exe'
        GrantedAccess: '0x1fffff'
    filter_optional_adobe_arm_helper:
        SourceImage|startswith:  # Example path: 'C:\Program Files (x86)\Common Files\Adobe\ARM\1.0\Temp\2092867405\AdobeARMHelper.exe'
            - 'C:\Program Files\Common Files\Adobe\ARM\'
            - 'C:\Program Files (x86)\Common Files\Adobe\ARM\'
        SourceImage|endswith: '\AdobeARMHelper.exe'
        GrantedAccess: '0x1410'
    condition: selection and not 1 of filter_optional_*
fields:
    - User
    - SourceImage
    - GrantedAccess
falsepositives:
    - Updaters and installers are typical false positives. Apply custom filters depending on your environment
level: medium
```

Notice how the condition filters out false positives (selection and not 1 of filter_optional_*).

### Example 2: Multiple Failed Logins From Single Source (Based on Event 4776)

According to Microsoft, Event 4776 generates every time that a credential validation occurs using NTLM authentication.

This event occurs only on the computer that is authoritative for the provided credentials. For domain accounts, the domain controller is authoritative. For local accounts, the local computer is authoritative.

It shows successful and unsuccessful credential validation attempts.

It shows only the computer name (Source Workstation) from which the authentication attempt was performed (authentication source). For example, if you authenticate from CLIENT-1 to SERVER-1 using a domain account you'll see CLIENT-1 in the Source Workstation field. Information about the destination computer (SERVER-1) isn't presented in this event.

If a credential validation attempt fails, you'll see a Failure event with Error Code parameter value not equal to `0x0`.

`lab_events_2.evtx` inside the `C:\Events\YARASigma` directory of this section's target contains events related to multiple failed login attempts against NOUSER (thanks to mdecrevoisier).

<img width="811" height="1088" alt="image" src="https://github.com/user-attachments/assets/9f57f253-fea2-4d24-a797-ed3de5718db2" />

<img width="808" height="1075" alt="image" src="https://github.com/user-attachments/assets/6fae84c8-4bdf-4c4b-9f38-96b3dae4f2a8" />

A valid Sigma rule to detect multiple failed login attempts originating from the same source can be found inside the `C:\Rules\sigma` directory of this section's target, saved as `win_security_susp_failed_logons_single_source2.yml`

<img width="723" height="663" alt="image" src="https://github.com/user-attachments/assets/8fb654c1-596f-4c0a-b738-cd0a4ef9a1ae" />

#### Sigma Rule Breakdown

- **`logsource`:** This section specifies that the rule is intended for Windows systems (`product: windows`) and focuses only on Security event logs (`service: security`).

<img width="633" height="113" alt="image" src="https://github.com/user-attachments/assets/535719f6-2457-45cc-9ab9-b8e14704d2af" />

- **`detection`:** `selection2` is essentially the filter. It's looking for logs with `EventID 4776` (`EventID: 4776`) regardless of the `TargetUserName` or `Workstation` values (`TargetUserName: '*'`, `Workstation: '*'`). `condition` counts instances of `TargetUserName` grouped by `Workstation` and checks if a workstation has more than three failed login attempts.

## Sigma Rule Development Resources

As you can imagine, the best Sigma rule development resource is the official documentation, which can be found at the following links.

- https://github.com/SigmaHQ/sigma/wiki/Rule-Creation-Guide
- https://github.com/SigmaHQ/sigma-specification

The following series of articles is the next best resource on Sigma rule developement.

- https://tech-en.netlify.app/articles/en510480/
- https://tech-en.netlify.app/articles/en513032/
- https://tech-en.netlify.app/articles/en515532/

In the upcoming section, we'll explore how to use Sigma rules for large-scale analysis of event logs with the aim of swiftly identifying threats.

## 🔑 Key Takeaway

Manual Sigma rule development involves identifying relevant event fields, defining detection conditions, and accounting for false positives. The LSASS example demonstrates how to detect suspicious process access using Sysmon Event ID 10, while the Event 4776 example shows how Sigma can identify repeated failed NTLM authentication attempts from a single source.
