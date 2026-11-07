# Hunting Evil with Sigma (Chainsaw Edition)

## Introduction

In cybersecurity, time is of the essence. Rapid analysis allows us to not just identify but also respond to threats before they escalate.

When we're up against the clock, racing to find a needle in a haystack of Windows Event Logs without access to a SIEM, Sigma rules combined with tools like Chainsaw and Zircolite are our best allies.

Both tools allow us to use Sigma rules to scan not just one, but multiple EVTX files concurrently, offering a broader and more comprehensive scan in a very efficient manner.

Chainsaw is a freely available tool designed to swiftly pinpoint security threats within Windows Event Logs. This tool enables efficient keyword-based event log searches and is equipped with integrated support for Sigma detection rules as well as custom Chainsaw rules. Therefore, it serves as a valuable asset for validating our Sigma rules by applying them to actual event logs. Let's download the Chainsaw from the official Github repository and run it with some sigma rules:

Chainsaw can be found inside the `C:\Tools\chainsaw` directory of this section's target.

Let's first run Chainsaw with `-h` flag to see the help menu.

<img width="740" height="771" alt="image" src="https://github.com/user-attachments/assets/2cc0969b-cff4-4272-8756-913d52baf53f" />

## Example 1: Hunting for Multiple Failed Logins From Single Source With Sigma

Let's put Chainsaw to work by applying our most recent Sigma rule, `win_security_susp_failed_logons_single_source2.yml` (available at `C:\Rules\sigma`), to `lab_events_2.evtx` (available at `C:\Events\YARASigma\lab_events_2.evtx`) that contains multiple failed login attempts from the same source.

<img width="657" height="749" alt="image" src="https://github.com/user-attachments/assets/6ba9a49d-1046-45f3-9489-2d77611ee04c" />

Our Sigma rule was able to identify the multiple failed login attempts against NOUSER.

Using the `-s` parameter, we can specify a directory containing Sigma detection rules (or one Sigma detection rule) and Chainsaw will automatically load, convert and run these rules against the provided event logs. The mapping file (specified through the `--mapping` parameter) tells Chainsaw which fields in the event logs to use for rule matching.

## Example 2: Hunting for Abnormal PowerShell Command Line Size With Sigma (Based on Event ID 4688)

Firstly, let's set the stage by recognizing that PowerShell, being a highly flexible scripting language, is an attractive target for attackers. Its deep integration with Windows APIs and .NET Framework makes it an ideal candidate for a variety of post-exploitation activities.

To conceal their actions, attackers utilize complex encoding layers or misuse cmdlets for purposes they weren't designed for. This leads to abnormally long PowerShell commands that often incorporate Base64 encoding, string merging, and several variables containing fragmented parts of the command.

A Sigma rule that can detect abnormally long PowerShell command lines can be found inside the `C:\Rules\sigma` directory of this section's target, saved as `proc_creation_win_powershell_abnormal_commandline_size.yml`

<img width="625" height="646" alt="image" src="https://github.com/user-attachments/assets/f4b44c5e-2c54-4ce3-82b6-3cf3c2b2ef93" />

### Sigma Rule Breakdown

- **logsource:** The rule looks into logs under the category of `process_creation` and is designed to work against Windows machines.

<img width="660" height="89" alt="image" src="https://github.com/user-attachments/assets/ef920c4a-e588-4a80-b12c-8d31b0addb81" />

- **detection:** The selection section checks if any Windows events with ID 4688 exist and also checks if the NewProcessName field ends with `\powershell.exe`, `\pwsh.exe`, or `\cmd.exe`. The selection_powershell section checks if the executed command line includes PowerShell-related executables and finally, the selection_length section checks if the CommandLine field of the 4688 event contains 1,000 characters or more. The condition section checks if the selection criteria inside the selection, selection_powershell, and selection_length sections are all met.

<img width="658" height="305" alt="image" src="https://github.com/user-attachments/assets/a490d471-1f49-4931-8b9e-522b7e6a1579" />

Let's put Chainsaw to work by applying the abovementioned Sigma rule, `proc_creation_win_powershell_abnormal_commandline_size.yml` (available at `C:\Rules\sigma`), to `lab_events_3.evtx` (available at `C:\Events\YARASigma\lab_events_3.evtx`, thanks to mdecrevoisier) that contains 4688 events with abnormally long PowerShell commands.

<img width="1050" height="1109" alt="image" src="https://github.com/user-attachments/assets/965c4c4d-2c7f-4fb8-ae85-ec7c1e87fce7" />

<img width="666" height="444" alt="image" src="https://github.com/user-attachments/assets/34de5845-9d19-4c67-b1dd-c16b93ba8cae" />

Our Sigma doesn't seem to be able to identify the abnormally long PowerShell commands within these 4688 events.

Does this mean that our Sigma rule is flawed? No! As discussed previously, Chainsaw's mapping file (specified through the `--mapping` parameter) tells it which fields in the event logs to use for rule matching.

It looks like the NewProcessName field was missing from the `sigma-event-logs-all.yml` mapping file.

We introduced the NewProcessName field into a `sigma-event-logs-all-new.yml` mapping file inside the `C:\Tools\chainsaw\mappings` directory of this section's target.

Let's run Chainsaw again with this new mapping file.

<img width="663" height="848" alt="image" src="https://github.com/user-attachments/assets/eece7cf9-842f-4807-8493-a0f846721b96" />

<img width="657" height="807" alt="image" src="https://github.com/user-attachments/assets/203c5226-7012-4d9d-81ec-5ca17e4360e0" />

Our Sigma rule successfully uncovered all three abnormally long PowerShell commands that exist inside `lab_events_3.evtx`

Remember that configuration when it comes to using or translating Sigma rules is of paramount importance!

## 🔑 Key Takeaway

Chainsaw enables efficient hunting across Windows Event Logs by applying Sigma rules to EVTX files. Accurate field mapping is essential for successful rule matching: if required fields such as `NewProcessName` are missing from the mapping file, valid Sigma rules may fail to detect events. Updating the mapping configuration can restore the expected detections.
