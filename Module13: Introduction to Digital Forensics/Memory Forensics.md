# Memory Forensics

## Memory Forensics Definition & Process

Memory forensics, also known as volatile memory analysis, is a specialized branch of digital forensics that focuses on the examination and analysis of the volatile memory (RAM) of a computer or digital device. Unlike traditional digital forensics, which involves analyzing data stored on non-volatile storage media like hard drives or solid-state drives, memory forensics deals with the live state of a system at a particular moment in time.

Here are some types of data found in RAM that are valuable for incident investigation:

- Network connections
- File handles and open Files
- Open registry keys
- Running processes on the system
- Loaded modules
- Loaded device drivers
- Command history and console sessions
- Kernel data structures
- User and credential information
- Malware artifacts
- System configuration
- Process memory regions

As we discussed both in the previous section and in the YARA & Sigma for SOC Analysts module, when malware operates, it often leaves traces or footprints in a system's active memory. By analyzing this memory, investigators can uncover malicious processes, identify indicators of compromise, and reconstruct the malware's actions.

It should be noted that in some cases, important data or encryption keys may reside in memory. Memory forensics can help recover this data, which may be crucial for an investigation.

The following outlines a systematic approach to memory forensics, formulated to aid in in-memory investigations and drawing inspiration from SANS's six-step memory forensics methodology.

### Process Identification and Verification

Let's begin by identifying all active processes. Malicious software often masquerades as legitimate processes, sometimes with subtle name variations to avoid detection. We need to:

- Enumerate all running processes.
- Determine their origin within the operating system.
- Cross-reference with known legitimate processes.
- Highlight any discrepancies or suspicious naming conventions.

### Deep Dive into Process Components

Once we've flagged potentially rogue processes, our next step is to scrutinize the associated Dynamic Link Libraries (DLLs) and handles. Malware often exploits DLLs to conceal its activities. We should:

- Examine DLLs linked to the suspicious process.
- Check for unauthorized or malicious DLLs.
- Investigate any signs of DLL injection or hijacking.

### Network Activity Analysis

Many malware strains, especially those that operate in stages, necessitate internet connectivity. They might beacon to Command and Control (C2) servers or exfiltrate data. To uncover these:

- Review active and passive network connections in the system's memory.
- Identify and document external IP addresses and associated domains.
- Determine the nature and purpose of the communication.

### Validate the process' legitimacy.

- Assess if the process typically requires network communication.
- Trace back to the parent process.
- Evaluate its behavior and necessity.

### Code Injection Detection

Advanced adversaries often employ techniques like process hollowing or utilize unmapped memory sections. To counter this, we should:

- Use memory analysis tools to detect anomalies or signs of these techniques.
- Identify any processes that seem to occupy unusual memory spaces or exhibit unexpected behaviors.

### Rootkit Discovery

Achieving stealth and persistence is a common goal for adversaries. Rootkits, which embed deep within the OS, grant threat actors continuous, often elevated, system access while evading detection. To tackle this:

- Scan for signs of rootkit activity or deep OS alterations.
- Identify any processes or drivers operating at unusually high privileges or exhibiting stealth behaviors.

### Extraction of Suspicious Elements

After pinpointing suspicious processes, drivers, or executables, we need to isolate them for in-depth analysis. This involves:

- Dumping the suspicious components from memory.
- Storing them securely for subsequent examination using specialized forensic tools.

## The Volatility Framework

The preferred tool for conducting memory forensics is Volatility. Volatility is a leading open-source memory forensics framework. At the heart of this framework lies the Volatility Python script. This script harnesses a plethora of plugins, enabling it to dissect memory images with precision. Given its Python foundation, we can execute Volatility on any platform that's Python-compatible. Moreover, our team can leverage Volatility to scrutinize memory image files from a broad spectrum of widely-used operating systems. This includes Windows, spanning from Windows XP to Windows Server 2016, macOS, and, of course, prevalent Linux distributions.

Volatility modules or plugins are extensions or add-ons that enhance the functionality of the Volatility Framework by extracting specific information or perform specific analysis tasks on memory images.

Some commonly used modules include:

| Module | Description |
|---|---|
| `pslist` | Lists the running processes. |
| `cmdline` | Displays process command-line arguments |
| `netscan` | Scans for network connections and open ports. |
| `malfind` | Scans for potentially malicious code injected into processes. |
| `handles` | Scans for open handles |
| `svcscan` | Lists Windows services. |
| `dlllist` | Lists loaded DLLs (Dynamic-link Libraries) in a process. |
| `hivelist` | Lists the registry hives in memory. |

Volatility offers extensive documentation. You can find modules and their associated documentation using the following links:

- **Volatility v2:** https://github.com/volatilityfoundation/volatility/wiki/Command-Reference
- **Volatility v3:** https://volatility3.readthedocs.io/en/latest/index.html

A useful Volatility (v2 & v3) cheatsheet can be found here: https://blog.onfvp.com/post/volatility-cheatsheet/

## Volatility v2 Fundamentals

Let's now see a demonstration of utilizing Volatility v2 to analyze a memory dump saved as `Win7-2515534d.vmem` inside the `/home/htb-student/MemoryDumps` directory of this section's target.

Volatility's help section and available plugins can be seen as follows.

<img width="637" height="820" alt="image" src="https://github.com/user-attachments/assets/dbeabb3a-4e52-4d83-9efa-ce8d2dd15394" />

### Identifying the Profile

Profiles are essential for Volatility v2 to interpret the memory data correctly (profile identification has been enhanced in v3). To determine the profile that matches the operating system of the memory dump we can use the `imageinfo` plugin as follows.

<img width="724" height="518" alt="image" src="https://github.com/user-attachments/assets/f4eb4167-8f6a-424a-82f8-ab8998fdf8c5" />

### Identifying Running Processes

Let's see if the suggested Win7SP1x64 profile is correct by trying to list running process via the `pslist` plugin.

<img width="725" height="818" alt="image" src="https://github.com/user-attachments/assets/6529c9f4-9ee4-47ca-8d91-cc069451bd79" />

It should be noted that even if we specify another profile from the suggested list Volatility may still provide us with the correct output.

### Identifying Network Artifacts

The `netscan` plugin can be used to scan for network artifacts as follows.

<img width="817" height="787" alt="image" src="https://github.com/user-attachments/assets/af4e12f2-d9a3-4a8d-8198-7eafbe41b070" />

To find `_TCPT_OBJECT` structures using pool tag scanning, use the `connscan` command. This can find artifacts from previous connections that have since been terminated, in addition to the active ones.

### Identifying Injected Code

The `malfind` plugin can be used to identify and extract injected code and malicious payloads from the memory of a running process as follows.

<img width="808" height="839" alt="image" src="https://github.com/user-attachments/assets/a470b2a9-501c-4bfc-a60a-141afb51ffd3" />

### Identifying Handles

The `handles` plugin in Volatility is used for analyzing the handles (file and object references) held by a specific process within a memory dump. Understanding the handles associated with a process can provide valuable insights during incident response and digital forensics investigations, as it reveals the resources and objects a process is interacting with. Here's how to use the `handles` plugin.

<img width="818" height="817" alt="image" src="https://github.com/user-attachments/assets/ad1c76c2-32f0-435a-9fc5-cdb3ff20b6ec" />

<img width="806" height="558" alt="image" src="https://github.com/user-attachments/assets/77f91b4b-6772-4f3d-b1b3-c52c51eba226" />

<img width="816" height="308" alt="image" src="https://github.com/user-attachments/assets/7bea7ea2-339b-4d5b-9e47-2463b4282fd6" />

### Identifying Windows Services

The `svcscan` plugin in Volatility is used for listing and analyzing Windows services running on a system within a memory dump. Here's how to use the `svcscan` plugin.

<img width="530" height="770" alt="image" src="https://github.com/user-attachments/assets/9660fbb7-5853-4fce-a722-f1d0fecd7c48" />

### Identifying Loaded DLLs

The `dlllist` plugin in Volatility is used for listing the dynamic link libraries (DLLs) loaded into the address space of a specific process within a memory dump. Here's how to use the `dlllist` plugin.

<img width="648" height="815" alt="image" src="https://github.com/user-attachments/assets/e374cf67-8817-47d6-86cf-3b9ef307c216" />

### Identifying Hives

The `hivelist` plugin in Volatility is used for listing the hives (registry files) present in the memory dump of a Windows system. Here's how to use the `hivelist` plugin.

<img width="719" height="572" alt="image" src="https://github.com/user-attachments/assets/68581fbc-3784-40ee-ae0d-591955f6b229" />

## Rootkit Analysis with Volatility v2

Let's now see a demonstration of utilizing Volatility v2 to analyze a memory dump saved as `rootkit.vmem` inside the `/home/htb-student/MemoryDumps` directory of this section's target.

### Understanding the EPROCESS Structure

EPROCESS is a data structure in the Windows kernel that represents a process. Each running process in the Windows operating system has a corresponding EPROCESS block in kernel memory. During memory analysis, the examination of EPROCESS structures is crucial for understanding the running processes on a system, identifying parent-child relationships, and determining which processes were active at the time of the memory capture.

<img width="1636" height="732" alt="image" src="https://github.com/user-attachments/assets/61a5ad11-f1cc-4424-af9a-41a6bfd9162a" />

### FLINK and BLINK

A doubly-linked list is a fundamental data structure in computer science and programming. It is a type of linked list where each node (record) contains two references or pointers:

- **Next Pointer:** This points to the next node in the list, allowing us to traverse the list in a forward direction.
- **Previous Pointer:** This points to the previous node in the list, allowing us to traverse the list in a backward direction.

Within the EPROCESS structure, we have ActiveProcessLinks as the doubly-linked list which contains the flink field and the blink field.

- **flink:** Is a forward pointer that points to the ActiveProcessLinks list entry of the _next_ EPROCESS structure in the list of active processes.
- **blink:** Is a backward pointer within the EPROCESS structure that points to the ActiveProcessLinks list entry of the _previous_ EPROCESS structure in the list of active processes.

These linked lists of EPROCESS structures are used by the Windows kernel to quickly iterate through all running processes on the system. The below diagram shows how this linked list looks like.

<img width="1869" height="453" alt="image" src="https://github.com/user-attachments/assets/b591a2ae-e7b1-4568-9f32-d395e584072c" />

### Identifying Rootkit Signs

Direct Kernel Object Manipulation (DKOM) is a sophisticated technique used by rootkits and advanced malware to manipulate the Windows operating system's kernel data structures in order to hide malicious processes, drivers, files, and other artifacts from detection by security tools and utilities running in userland (i.e., in user mode).

If, for example, a monitoring tool is dependent on the EPROCESS structure for the enumeration of the running processes, and there's a rootkit running on the system which manipulates the EPROCESS structure directly in kernel memory by altering the EPROCESS structure or unlinking a process from lists, the monitoring tool will not be able to get the hidden process in the currently running processes list.

The below screenshot shows a graphical representation of how this unlinking actually works.

<img width="1877" height="834" alt="image" src="https://github.com/user-attachments/assets/6e58eed5-d6c8-4216-87d6-d27442a42a4b" />

The `psscan` plugin is used to enumerate running processes. It scans the memory pool tags associated with each process's EPROCESS structure. This technique can help identify processes that may have been hidden or unlinked by rootkits, as well as processes that have been terminated but have not been removed from memory yet. This plugin can be used as follows.

<img width="732" height="780" alt="image" src="https://github.com/user-attachments/assets/820e0d77-5a01-4635-ab3b-10ced1123dd4" />

In the output below, we can see that the `pslist` plugin could not find `test.exe` which was hidden by a rootkit, but the `psscan` plugin was able to find it.

<img width="725" height="855" alt="image" src="https://github.com/user-attachments/assets/c227d1c9-7556-4358-b8d4-933ff5060322" />

## Memory Analysis Using Strings

Analyzing strings in memory dumps is a valuable technique in memory forensics and incident response. Strings often contain human-readable information, such as text messages, file paths, IP addresses, and even passwords.

We can either use the Strings tool from the Sysinternals suite if our system is Windows-based, or the `strings` command from Binutils, if our system is Linux-based.

Let's see some examples against a memory dump named `Win7-2515534d.vmem` that resides in the `/home/htb-student/MemoryDumps` directory of this section's target.

### Identifying IPv4 Addresses

<img width="722" height="230" alt="image" src="https://github.com/user-attachments/assets/c0498438-221f-4a85-9948-d3969e457ece" />

### Identifying Email Addresses

<img width="734" height="856" alt="image" src="https://github.com/user-attachments/assets/9b57190f-70d7-405f-a201-35403c30a71f" />

### Identifying Command Prompt or PowerShell Artifacts

<img width="734" height="350" alt="image" src="https://github.com/user-attachments/assets/1dd4ac61-a648-4e6e-9f61-aeb2e15a2bc3" />

These are just a few examples of common string searches during memory forensics and incident response. You can adapt and customize these searches based on your specific investigation's needs and the types of information you are looking for in the memory dump. Regular expressions can be powerful tools for pattern matching and data extraction during forensic analysis.

## 🔑 Key Takeaway

Memory forensics examines volatile RAM to uncover active processes, network connections, loaded modules, malware artifacts, and other evidence that may not exist on disk. The Volatility Framework provides plugins such as `pslist`, `netscan`, `malfind`, `handles`, `svcscan`, `dlllist`, `hivelist`, and `psscan` to investigate memory images, detect suspicious activity, and identify processes hidden by rootkits. String analysis can also reveal useful indicators such as IP addresses, email addresses, and command-line artifacts.
