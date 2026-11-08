# Hunting Evil with Sigma (Splunk Edition)

## Introduction

As discussed when introducing Sigma, Sigma rules revolutionize our approach to log analysis and threat detection. What we're dealing with here is a sort of Rosetta Stone for SIEM systems. Sigma is like a universal translator that brings in a level of abstraction to event logs, taking away the painful element of SIEM-specific query languages.

Let's validate this assertion by converting two Sigma rules into their corresponding SPL formats and examining the outcomes.

## Example 1: Hunting for MiniDump Function Abuse to Dump LSASS's Memory (comsvcs.dll via rundll32)

A Sigma rule named `proc_access_win_lsass_dump_comsvcs_dll.yml` can be found inside the `C:\Tools\chainsaw\sigma\rules\windows\process_access` directory of the previous section's target.

This Sigma rule detects adversaries leveraging the MiniDump export function of `comsvcs.dll` via `rundll32` to perform a memory dump from LSASS.

We can translate this rule into a Splunk search with `sigmac` (available at `C:\Tools\sigma-0.21\tools`) as follows.

<img width="820" height="119" alt="image" src="https://github.com/user-attachments/assets/a607592f-7d55-4d0c-815f-293420d02efe" />

<img width="1777" height="1451" alt="image" src="https://github.com/user-attachments/assets/8c12a99a-6e35-47e0-8d8e-ce226103ca42" />

The Splunk search provided by `sigmac` was indeed able to detect MiniDump function abuse to dump LSASS's memory.

## Example 2: Hunting for Notepad Spawning Suspicious Child Process

A Sigma rule named `proc_creation_win_notepad_susp_child.yml` can be found inside the `C:\Rules\sigma` directory of the previous section's target.

This Sigma rule detects `notepad.exe` spawning a suspicious child process.

We can translate this rule into a Splunk search with `sigmac` (available at `C:\Tools\sigma-0.21\tools`) as follows.

<img width="811" height="164" alt="image" src="https://github.com/user-attachments/assets/2547f4cd-74e5-4103-b7fa-68e55c291d97" />

<img width="1779" height="1467" alt="image" src="https://github.com/user-attachments/assets/f920447e-a97a-487d-996f-a51324e5c8dd" />

The Splunk search provided by `sigmac` was indeed able to detect `notepad.exe` spawning suspicious processes (such as PowerShell).

## Configuration Note

Please note that more frequently than not you will have to tamper with Sigma's config files (available inside the `C:\Tools\sigma-0.21\tools\config` directory of the previous section's target) in order for the SIEM queries to be readily usable.

## 🔑 Key Takeaway

Sigma provides a universal format for writing detection rules that can be translated into SIEM-specific queries such as Splunk SPL. Tools like `sigmac` make this conversion possible, but Sigma configuration files may need to be adjusted to ensure the generated queries are usable in the target SIEM.
