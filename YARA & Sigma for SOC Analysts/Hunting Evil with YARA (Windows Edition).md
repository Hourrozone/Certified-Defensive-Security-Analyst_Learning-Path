<img width="744" height="190" alt="image" src="https://github.com/user-attachments/assets/b4e0ac57-875c-419d-b426-66bc6c14d685" /># Hunting Evil with YARA (Windows Edition)

## Introduction

In this section, we'll explore using YARA on Windows systems for identifying threats both on disk and in memory.

## Hunting for Malicious Executables on Disk with YARA

As we saw in the previous section, YARA is a potent weapon in the arsenal of cybersecurity professionals for detecting and hunting malicious executables on a disk. With custom YARA rules or established ones at our disposal, we can pinpoint suspicious or potentially malicious files based on distinct patterns, traits, or behaviors.

We will be using a sample that we analyzed previously named `dharma_sample.exe` residing in the `C:\Samples\YARASigma` directory of this section's target.

We'll first examine the malware sample inside a hex editor (HxD, located at `C:\Program Files\HxD`) to identify the previously discovered string `C:\crysis\Release\PDB\payload.pdb`.

<img width="2857" height="1100" alt="image" src="https://github.com/user-attachments/assets/90d37c2f-3b08-4275-8078-09f89070dd26" />

<img width="2854" height="1097" alt="image" src="https://github.com/user-attachments/assets/f268c14a-573a-4d12-b3dd-96cc7b91ca6c" />

If we scroll almost to the bottom, we will notice yet another seemingly unique `sssssbsss` string.

<img width="2854" height="1099" alt="image" src="https://github.com/user-attachments/assets/48ae1dbb-a967-4259-a86e-d98ce5f0cab2" />

Going forward, we will craft a rule grounded in these patterns and then utilize the YARA utility to scour the filesystem for similar executables.

**Note:** In a Linux machine the `hexdump` utility could have been used to identify the aforementioned hex bytes as follows.

<img width="744" height="190" alt="image" src="https://github.com/user-attachments/assets/04246ac1-94cb-418a-b8d6-817ba4043bf8" />

<img width="735" height="179" alt="image" src="https://github.com/user-attachments/assets/7780badc-7a8c-41db-8933-e4c9aa623e44" />

Let's incorporate all identified hex bytes into a rule, enhancing our ability to detect this string across any disk-based executable.

<img width="740" height="354" alt="image" src="https://github.com/user-attachments/assets/a2db33ba-b698-4f85-b301-73e5effbb4f6" />

This rule (`dharma_ransomware.yar`) can be found inside the `C:\Rules\yara` directory of this section's target.

Initiating the YARA executable with this rule, let's observe if it highlights other analogous samples on the disk.

<img width="737" height="481" alt="image" src="https://github.com/user-attachments/assets/53618de4-a383-46e7-9d46-cab68d81b711" />

### Command Breakdown

- **`yara64.exe`:** Refers to the YARA64 executable, which is the YARA scanner specifically designed for 64-bit systems.
- **`-s C:\Rules\yara\dharma_ransomware.yar`:** Specifies the YARA rules file to be used for scanning. In this case, the rules file named `dharma_ransomware.yar` located in the `C:\Rules\yara` directory is provided.
- **`C:\Samples\YARASigma`:** Specifies the path or directory to be scanned by YARA. In this case, the directory being scanned is `C:\Samples\YARASigma`.
- **`-r`:** Indicates that the scanning operation should be performed recursively, meaning YARA will scan files within subdirectories of the specified directory as well.
- **`2>nul`:** Redirects the error output (stream 2) to a null device, effectively hiding any error messages that might occur during the scanning process.

As we can see, the `pdf_reader.exe`, `microsoft.com`, `check_updates.exe`, and `KB5027505.exe` files are detected by this rule (in addition to `dharma_sample.exe` of course).

Now, let's pivot, applying YARA rules to live processes.

## Hunting for Evil Within Running Processes with YARA

To ascertain if malware lurks in ongoing processes, we'll unleash the YARA scanner on the system's active processes. Let's demonstrate using a YARA rule that targets Metasploit's meterpreter shellcode, believed to be lurking in a running process.

**YARA Rule Source:** https://github.com/cuckoosandbox/community/blob/master/data/yara/shellcode/metasploit.yar

<img width="739" height="384" alt="image" src="https://github.com/user-attachments/assets/da2680c4-1be9-491b-884f-d2c37b175761" />

We will be using a sample that we analyzed previously named `htb_sample_shell.exe` residing in the `C:\Samples\YARASigma` directory of this section's targe

`htb_sample_shell.exe` injects Metasploit's meterpreter shellcode into the `cmdkey.exe` process. Let's activate it, ensuring successful injection.

**Note:** Make sure you launch PowerShell as an administrator.

<img width="735" height="259" alt="image" src="https://github.com/user-attachments/assets/9d31b057-d605-4db0-a769-9ac9a0690619" />

With the injection executed, let's scan every active system process as follows, through another PowerShell terminal (Run as administrator).

<img width="739" height="772" alt="image" src="https://github.com/user-attachments/assets/e2935967-cc7b-4043-91bf-331f02503324" />

We're leveraging a concise PowerShell script. The `Get-Process` command fetches running processes, and with the help of the pipe symbol (`|`), this data funnels into the script block (`{...}`). Here, `ForEach-Object` dissects each process, prompting `yara64.exe` to apply our YARA rule on each process's memory.

Let's observe if the child process (PID `9084`) gets flagged.

From the results, the meterpreter shellcode seems to have infiltrated a process with PID `9084`. We can also guide the YARA scanner with a specific PID as follows.

<img width="721" height="656" alt="image" src="https://github.com/user-attachments/assets/52f29650-5db3-4b8a-b9a2-217782f6a23a" />

The screenshot below shows an overview of what we discussed above.

<img width="2453" height="1473" alt="image" src="https://github.com/user-attachments/assets/114dd426-f523-4534-8dc9-5195c3fb7fd6" />

## Hunting for Evil Within ETW Data with YARA

In the module titled Windows Event Logs & Finding Evil we explored ETW and introduced SilkETW. In this section, we'll circle back to ETW data, highlighting how YARA can be used to filter or tag certain events.

A quick recap first. According to Microsoft, Event Tracing For Windows (ETW) is a general-purpose, high-speed tracing facility provided by the operating system. Using a buffering and logging mechanism implemented in the kernel, ETW provides a tracing mechanism for events raised by both user-mode applications and kernel-mode device drivers.

<img width="1583" height="842" alt="image" src="https://github.com/user-attachments/assets/f2557f2f-79f3-494e-9907-51fee84d744d" />

### ETW Components

- **Controllers:** Controllers possess functionalities that encompass initiating and terminating trace sessions. They also have the capability to enable or disable providers within a specific trace.
- **Providers:** Providers are crucial, as they generate events and channel them to the designated ETW sessions.
- **Consumers:** Consumers are the subscribers to specific events. They tap into these events and then receive them for in-depth processing or analysis.

### Useful Providers

- **Microsoft-Windows-Kernel-Process:** This ETW provider is instrumental in monitoring process-related activity within the Windows kernel. It can aid in detecting unusual process behaviors such as process injection, process hollowing, and other tactics commonly used by malware and advanced persistent threats (APTs).
- **Microsoft-Windows-Kernel-File:** As the name suggests, this provider focuses on file-related operations. It can be employed for detection scenarios involving unauthorized file access, changes to critical system files, or suspicious file operations indicative of exfiltration or ransomware activity.
- **Microsoft-Windows-Kernel-Network:** This ETW provider offers visibility into network-related activity at the kernel level. It's especially useful in detecting network-based attacks such as data exfiltration, unauthorized network connections, and potential signs of command and control (C2) communication.
- **Microsoft-Windows-SMBClient/SMBServer:** These providers monitor Server Message Block (SMB) client and server activity, providing insights into file sharing and network communication. They can be used to detect unusual SMB traffic patterns, potentially indicating lateral movement or data exfiltration.
- **Microsoft-Windows-DotNETRuntime:** This provider focuses on .NET runtime events, making it ideal for identifying anomalies in .NET application execution, potential exploitation of .NET vulnerabilities, or malicious .NET assembly loading.
- **OpenSSH:** Monitoring the OpenSSH ETW provider can provide important insights into Secure Shell (SSH) connection attempts, successful and failed authentications, and potential brute force attacks.
- **Microsoft-Windows-VPN-Client:** This provider enables tracking of Virtual Private Network (VPN) client events. It can be useful for identifying unauthorized or suspicious VPN connections.
- **Microsoft-Windows-PowerShell:** This ETW provider tracks PowerShell execution and command activity, making it invaluable for detecting suspicious PowerShell usage, script block logging, and potential misuse or exploitation.
- **Microsoft-Windows-Kernel-Registry:** This provider monitors registry operations, making it useful for detection scenarios related to changes in registry keys, often associated with persistence mechanisms, malware installation, or system configuration changes.
- **Microsoft-Windows-CodeIntegrity:** This provider monitors code and driver integrity checks, which can be key in identifying attempts to load unsigned or malicious drivers or code.
- **Microsoft-Antimalware-Service:** This ETW provider can be employed to detect potential issues with the antimalware service, including disabled services, configuration changes, or potential evasion techniques employed by malware.
- **WinRM:** Monitoring the Windows Remote Management (WinRM) provider can reveal unauthorized or suspicious remote management activity, often indicative of lateral movement or remote command execution.
- **Microsoft-Windows-TerminalServices-LocalSessionManager:** This provider tracks local Terminal Services sessions, making it useful for detecting unauthorized or suspicious remote desktop activity.
- **Microsoft-Windows-Security-Mitigations:** This provider keeps tabs on the effectiveness and operations of security mitigations in place. It's essential for identifying potential bypass attempts of these security controls.
- **Microsoft-Windows-DNS-Client:** This ETW provider gives visibility into DNS client activity, which is crucial for detecting DNS-based attacks, including DNS tunneling or unusual DNS requests that may indicate C2 communication.
- **Microsoft-Antimalware-Protection:** This provider monitors the operations of antimalware protection mechanisms. It can be used to detect any issues with these mechanisms, such as disabled protection features, configuration changes, or signs of evasion techniques employed by malicious actors.

## YARA Rule Scanning on ETW (Using SilkETW)

SilkETW is an open-source tool to work with Event Tracing for Windows (ETW) data. SilkETW provides enhanced visibility and analysis of Windows events for security monitoring, threat hunting, and incident response purposes. The best part of SilkETW is that it also has an option to integrate YARA rules. It includes YARA functionality to filter or tag event data.

SilkETW resides in the `C:\Tools\SilkETW\v8\SilkETW` directory of this section's target.

<img width="654" height="813" alt="image" src="https://github.com/user-attachments/assets/7fe62653-a292-471b-8702-d729640c8435" />

The help menu provides many examples of how we can use the tool. Let's experiment with some of the YARA scanning options on a few ETW providers.

### Example 1: YARA Rule Scanning on Microsoft-Windows-PowerShell ETW Data

The command below executes the SilkETW tool with specific options to perform event tracing and analysis on PowerShell-related events in Windows.

**Note:** Make sure you launch PowerShell as an administrator.

<img width="658" height="295" alt="image" src="https://github.com/user-attachments/assets/5b7b0f90-3139-4c6c-a6d2-5ae6d42a5f61" />

#### Command Breakdown

- **`-t user`:** Specifies the event tracing mode. In this case, it is set to "user," indicating that the tool will trace user-mode events (events generated by user applications).
- **`-pn Microsoft-Windows-PowerShell`:** Specifies the name of the provider or event log that you want to trace. In this command, it targets events from the "Microsoft-Windows-PowerShell" provider, which is responsible for generating events related to PowerShell activity.
- **`-ot file`:** Specifies the output format for the collected event data. In this case, it is set to "file," meaning that the tool will save the event data to a file.
- **`-p ./etw_ps_logs.json`:** Specifies the output file path and filename. The tool will save the collected event data in JSON format to a file named "etw_ps_logs.json" in the current directory.
- **`-l verbose`:** Sets the logging level to "verbose." This option enables more detailed logging information during the event tracing and analysis process.
- **`-y C:\Rules\yara`:** Enables YARA scanning and specifies a path containing YARA rules. This option indicates that the tool will perform YARA scanning on the collected event data.
- **`-yo Matches`:** Specifies the YARA output option. In this case, it is set to "Matches," meaning that the tool will display YARA matches found during the scanning process.

Inside the `C:\Rules\yara` directory of this section's target there is a YARA rules file named `etw_powershell_hello.yar` that looks for certain strings in PowerShell script blocks.

<img width="660" height="215" alt="image" src="https://github.com/user-attachments/assets/17c98871-6571-4fb0-86fc-6794840b4528" />

Let's now execute the following PowerShell command through another PowerShell terminal and see if it will get detected by SilkETW (where the abovementioned YARA rule has been loaded).

<img width="655" height="325" alt="image" src="https://github.com/user-attachments/assets/af2529d2-1836-4075-abca-d48464ad5c2e" />

We have a match!

<img width="659" height="291" alt="image" src="https://github.com/user-attachments/assets/ea0f091d-2e3c-4d70-a35a-4981766ce013" />

### Example 2: YARA Rule Scanning on Microsoft-Windows-DNS-Client ETW Data

The command below executes the SilkETW tool with specific options to perform event tracing and analysis on DNS-related events in Windows.

<img width="660" height="153" alt="image" src="https://github.com/user-attachments/assets/e2f6e99f-a984-4663-8f9e-a79338255904" />

Inside the `C:\Rules\yara` directory of this section's target there is a YARA rules file named `etw_dns_wannacry.yar` that looks for a hardcoded domain that exists in Wannacry ransomware samples in DNS events.

<img width="650" height="212" alt="image" src="https://github.com/user-attachments/assets/53baf857-d05a-4018-a268-d632c237cac1" />

Let's now execute the following command through another PowerShell terminal and see if it will get detected by SilkETW (where the abovementioned YARA rule has been loaded).


We have a match!

<img width="657" height="323" alt="image" src="https://github.com/user-attachments/assets/4848f7ff-f82e-40ad-9641-7daa58d45d38" />

## 🔑 Key Takeaway

YARA can be used on Windows to hunt for malicious files on disk, detect injected shellcode in running processes, and scan ETW event data through SilkETW. By combining custom or existing YARA rules with file, process-memory, and ETW scanning, analysts can identify suspicious patterns and behaviors across multiple areas of a Windows system.
