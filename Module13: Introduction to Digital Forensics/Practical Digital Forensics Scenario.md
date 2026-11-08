<img width="1569" height="251" alt="image" src="https://github.com/user-attachments/assets/f92bcf87-bd67-4cd3-aab6-9f55d6a25332" />
## Scenario Overview

You belong to the digital forensics team and are assigned to investigate an incident related to a Windows system using a memory dump, a full disk image, and rapid triage artifacts.

**Memory dump's location:** `C:\Users\johndoe\Desktop\memdump\PhysicalMemory.raw`

**Rapid triage artifacts' locations:**
- `C:\Users\johndoe\Desktop\kapefiles`
- `C:\Users\johndoe\Desktop\files`

**Full disk image's location:** `C:\Users\johndoe\Desktop\fulldisk.raw.001`

**Parsed full disk image data location:** `C:\Users\johndoe\Desktop\MalwareAttack`

### Notes

- When analyzing with Autopsy, we strongly suggest accessing the case from `C:\Users\johndoe\Desktop\MalwareAttack`.
- During an investigation, it's imperative to examine artifacts or evidence on a specialized system tailored for forensic tasks. However, for the sake of expediency, the analysis is conducted within the impacted system itself.

---

## Memory Analysis with Volatility v3

The affected system's memory dump resides in `C:\Users\johndoe\Desktop\memdump\PhysicalMemory.raw`.

### Identifying the Memory Dump's Profile

Let's start by obtaining OS & kernel details of the Windows memory sample being analyzed, leveraging Volatility's `windows.info` plugin.

<img width="736" height="620" alt="image" src="https://github.com/user-attachments/assets/eca383fa-151c-44a2-b7a9-c553af6eec77" />




### Identifying Injected Code

Volatility's `windows.malfind` plugin can then be used to list process memory ranges that potentially contain injected code as follows.

<img width="736" height="774" alt="image" src="https://github.com/user-attachments/assets/a6bf2c3c-4975-4bc5-a84f-44f938911639" />

<img width="734" height="808" alt="image" src="https://github.com/user-attachments/assets/4bf2926b-7ee7-4a2f-9ee8-ec3669b30b16" />

<img width="734" height="850" alt="image" src="https://github.com/user-attachments/assets/4238dbc3-df7c-49c9-8127-ed6b4b080605" />

When a process allocates a memory page with `PAGE_EXECUTE_READWRITE` permissions, it's essentially requesting the ability to both execute and write to that memory region. In layman's terms, the process is saying, "I want to be able to run code from here, but I also want the flexibility to change what that code is on the fly."

Now, why does that raise eyebrows? Well, legitimate applications typically segregate the tasks of code execution and data writing. They'll have specific regions of memory for running code (executable) and separate regions where data is written or modified. This separation is a fundamental security principle, ensuring that data isn't inadvertently executed or that executable regions aren't tampered with unexpectedly.

However, many types of malware, especially those that employ code injection techniques, require the ability to write their payload into memory and then execute it. By allocating memory with `PAGE_EXECUTE_READWRITE` permissions, they can write and subsequently execute malicious code within the same memory region, making their malicious activities more streamlined and efficient.

In essence, while not every instance of `PAGE_EXECUTE_READWRITE` is malicious, its presence is a strong indicator of potential malfeasance, and it's something we, as vigilant security analysts, should scrutinize closely.

### Identifying Running Processes

Let's now list the processes present in this particular Windows memory image through Volatility's `windows.pslist` plugin as follows.

<img width="748" height="844" alt="image" src="https://github.com/user-attachments/assets/747bf6e4-b354-416f-a92e-1557d3bab6a3" />

<img width="747" height="841" alt="image" src="https://github.com/user-attachments/assets/97176bcc-2382-4932-a181-967245899198" />

<img width="734" height="822" alt="image" src="https://github.com/user-attachments/assets/84828189-bac7-4ed7-9672-a27dd50815ea" />


If we want to list processes in a tree based on their parent process ID, we can do that through Volatility's `windows.pstree` plugin as follows.

<img width="728" height="797" alt="image" src="https://github.com/user-attachments/assets/1ac1ffdd-43d8-4e24-aad5-d68ab9735e75" />

<img width="745" height="798" alt="image" src="https://github.com/user-attachments/assets/ec0f4fe9-e3b0-4edf-8e02-0df7237cf169" />

<img width="730" height="859" alt="image" src="https://github.com/user-attachments/assets/732d8047-c2da-46cf-808b-3d65b1ec8ab8" />

<img width="735" height="831" alt="image" src="https://github.com/user-attachments/assets/5f2b07a1-4295-403c-9b02-6567afdcba75" />

### Identifying Process Command Lines

Volatility's `windows.cmdline` plugin can provide us with a list of process command line arguments as follows.

<img width="739" height="772" alt="image" src="https://github.com/user-attachments/assets/4fae5bfa-954f-4369-a555-8ce021eb7ab6" />

<img width="729" height="806" alt="image" src="https://github.com/user-attachments/assets/b12584fe-d24b-4a9a-9169-9c1102699424" />

<img width="734" height="775" alt="image" src="https://github.com/user-attachments/assets/504f4904-b83b-4c19-a9c1-b2c229057472" />

<img width="741" height="704" alt="image" src="https://github.com/user-attachments/assets/02564714-49e7-4844-8968-fdba8776e468" />

### Dumping Process Memory & Leveraging YARA

It should be obvious by now that process 3648 looks suspicious. To extract all memory resident pages in a process into an individual file we can use Volatility's `windows.memmap` plugin as follows.

<img width="715" height="849" alt="image" src="https://github.com/user-attachments/assets/4c5195e6-c572-4510-92a3-f1f3e7c04376" />

`pid.3648.dmp` can be found inside the `c:\Users\johndoe\Desktop` directory of this section's target for your convenience.

To glean more details about the process with ID 3648, we can employ YARA. By leveraging a PowerShell loop, we can systematically scan the process dump using all available rules of the https://github.com/Neo23x0/signature-base/tree/master YARA rules repository, which can be found inside the `C:\Users\johndoe\Desktop\yara-4.3.2-2150-win64\rules` directory of this section's target.

<img width="731" height="249" alt="image" src="https://github.com/user-attachments/assets/d3ccc270-c2b5-4c2e-b427-3b8d8f4fd24e" />

We notice some hits related to the Cobalt Strike framework.

### Identifying Loaded DLLs

Upon scrutinizing the command lines, we identified arguments pointing to `payload.dll` for process 3648, with the Start function serving as a clear sign of `payload.dll`'s execution. To further our understanding of the associated DLLs, we can employ Volatility's `windows.dlllist` plugin as follows.

<img width="732" height="856" alt="image" src="https://github.com/user-attachments/assets/3adfbf2c-ceed-4253-b53f-8271263e33c8" />

<img width="723" height="846" alt="image" src="https://github.com/user-attachments/assets/33d03d76-115b-4424-8c19-8d61b8fdcb9c" />

<img width="716" height="805" alt="image" src="https://github.com/user-attachments/assets/72831d78-7549-4710-bc76-d9e2a8c93a54" />

We notice `E:\payload.dll` in Volatility's output. Based on its location, we surmise it could originate from an external USB or perhaps a mounted ISO file. We'll earmark this DLL for a more in-depth analysis later on.

### Identifying Handles

Next, let's identify the files and registry entries accessed by the suspicious process using Volatility's `windows.handles` plugin.

When a process needs to read from or write to a file, it doesn't directly interact with the file's data on the disk. Instead, the process requests the operating system to open the file, and in return, the OS provides a file handle. This handle is essentially a ticket that grants the process permission to perform operations on that file. Every subsequent operation the process performs on that file - be it reading, writing, or closing - is done through this handle.

Open handles can be a goldmine for forensic analysts. By examining the list of open handles, we can determine which processes were accessing which files or registry keys at a particular point in time. This can provide insights into the behavior of potentially malicious software or the actions of a user.

<img width="727" height="779" alt="image" src="https://github.com/user-attachments/assets/d1ce973e-a5a5-4a89-bd3a-706ca7d5d89d" />

<img width="757" height="840" alt="image" src="https://github.com/user-attachments/assets/628bd1b2-df93-4c1c-8ca7-017592ec0808" />

<img width="750" height="735" alt="image" src="https://github.com/user-attachments/assets/942cdb41-f147-4a14-9ad4-c7e7c361008a" />


It's evident (based on `\Device\HarddiskVolume3\Users\johndoe\Desktop`) that the process has interactions with certain files located on the Desktop, which warrants a closer look shortly.

### Identifying Network Artifacts

Moving away from processes, Volatility's `windows.netstat` plugin can traverse network tracking structures to help us analyze connection details within a memory image.

<img width="705" height="843" alt="image" src="https://github.com/user-attachments/assets/2b3ab5af-23aa-4b17-9ee8-3f52d9733c6d" />

<img width="745" height="805" alt="image" src="https://github.com/user-attachments/assets/34a0df96-3a7b-45a0-852f-969078c457b3" />

For a more exhaustive network analysis, we can also employ Volatility's `windows.netscan` plugin as follows:

<img width="733" height="786" alt="image" src="https://github.com/user-attachments/assets/75708bec-14a4-4f57-9b90-1b725ecb129e" />

<img width="746" height="860" alt="image" src="https://github.com/user-attachments/assets/7a116915-adaa-41b9-8b55-e9435ba4a299" />

<img width="685" height="859" alt="image" src="https://github.com/user-attachments/assets/83538993-78f3-4a48-ab2f-a0280880d613" />

<img width="720" height="798" alt="image" src="https://github.com/user-attachments/assets/6d1b9bc7-d6cc-4203-aedc-86bccac42a56" />

The suspicious process (PID 3648) has been communicating with `44.214.212.249` over port 80.

---

## Disk Image/Rapid Triage Data Examination & Analysis

### Searching for Keywords with Autopsy

Let's first open Autopsy and access the case from `C:\Users\johndoe\Desktop\MalwareAttack`.

Now, to trace `payload.dll` on the disk, we'll navigate through Autopsy and initiate a search for the `payload.dll` keyword, prioritizing results by their creation time.

<img width="1432" height="658" alt="image" src="https://github.com/user-attachments/assets/4ad175ff-49f1-4283-8d75-17aef4bf4730" />

Among the 29 findings, the `Finance08062023.iso` file in the Downloads directory should pique our interest (recall the DLL on the E drive?). We can extract this file for subsequent scrutiny, by right-clicking and selecting Extract File(s).

<img width="842" height="510" alt="image" src="https://github.com/user-attachments/assets/9fd1611b-b28a-4e3b-8b4e-0b1bd6c09137" />

Given the file's presence in the Downloads folder and a corresponding Chrome cache file (`f_000003`) pointing to similar strings, it's plausible that the ISO file was fetched via a browser.

<img width="953" height="353" alt="image" src="https://github.com/user-attachments/assets/1fc80b76-4852-40f4-bf37-c7c417b61404" />

### Identifying Web Download Information & Extracting Files with Autopsy

To extract web download details, we'll harness the capabilities of ADS. Within Autopsy, we can access the Downloads directory to locate our file. Here, the `.Zone.Identifier` information, courtesy of the Alternate Data Stream (ADS) file attributes, is invaluable.

<img width="1404" height="348" alt="image" src="https://github.com/user-attachments/assets/afc205cf-a271-48bb-83a7-129b37c95809" />

This identifier reveals the file's internet origin, and we can pinpoint the HostUrl from which the malicious ISO was sourced.

<img width="968" height="224" alt="image" src="https://github.com/user-attachments/assets/35cd0649-07a3-4797-87e0-31e68f1fca7b" />

Corroborating our findings, Autopsy's Web Downloads artifacts confirm that `Finance08062023.iso` was sourced from `letsgohunt[.]site`.

<img width="1826" height="690" alt="image" src="https://github.com/user-attachments/assets/0f40385d-7d0a-4f53-8bfb-93fea729f83f" />

Upon mounting the extracted ISO file, we notice that it houses both a DLL and a shortcut file, which leverages `rundll32.exe` to activate `payload.dll`.

<img width="850" height="611" alt="image" src="https://github.com/user-attachments/assets/8d00ab6a-1904-4233-a716-014d93c6ede3" />

### Extracting Cobalt Strike Beacon Configuration

Given our knowledge of the attacker's use of Cobalt Strike, we can attempt to extract the beacon configuration via the CobaltStrikeParser script, that can found inside the `C:\Users\johndoe\Desktop\CobaltStrikeParser-master\CobaltStrikeParser-master` directory of this section's target as follows.

<img width="734" height="830" alt="image" src="https://github.com/user-attachments/assets/2441ee93-174c-4607-9ff8-661706480346" />

<img width="719" height="843" alt="image" src="https://github.com/user-attachments/assets/c05d2b44-8389-4491-b6b5-ff254b878b7e" />

### Identifying Persistence with Autoruns

For persistence mechanisms, let's inspect the `C:\Users\johndoe\Desktop\files\johndoe_autoruns.arn` file using the Autoruns tool.

Within the Logon section, we notice a LocalSystem entry with the following details:

- **Registry path:** `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
- **Image path:** `C:\ProgramData\svchost.exe`
- **Timestamp:** Thu Aug 10 11:25:51 2023 (this is a local timestamp, UTC: 09:25:51)

<img width="1419" height="331" alt="image" src="https://github.com/user-attachments/assets/c7012350-dd2d-4524-bc7d-0213a39471ba" />

Additionally, an odd `photo433.exe` executable has been flagged.

`photo433.exe` has been extracted during the Rapid Triage process and resides inside the `C:\Users\johndoe\Desktop\kapefiles\auto\C%3A\Users\johndoe\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\` directory of this section's target.

Its SHA256 hash can be identified by either using PowerShell as follows or through Autopsy.

<img width="744" height="222" alt="image" src="https://github.com/user-attachments/assets/e657e9e9-9d7c-4e05-af6f-6d5797b7ab10" />

<img width="1865" height="218" alt="image" src="https://github.com/user-attachments/assets/272e6a9b-d695-4f2c-8332-30090065a299" />

<img width="756" height="289" alt="image" src="https://github.com/user-attachments/assets/50e21893-bad6-456c-83f7-2bffddace023" />

For a comprehensive assessment let's submit this hash to VirusTotal

<img width="1896" height="879" alt="image" src="https://github.com/user-attachments/assets/23e7edc5-b2f9-4e97-ae60-ab994a30b468" />

By navigating to the Scheduled Tasks tab of the Autoruns tool, we uncover another persistence mechanism.

<img width="1413" height="275" alt="image" src="https://github.com/user-attachments/assets/009e3acc-0735-4cfe-9cbd-a5c72fd9f106" />

### Analyzing MFT Data with Autopsy

While using the Autoruns tool to search for persistence, we came across the image path `C:\ProgramData\svchost.exe`.

Let's dive into `C:\ProgramData` using Autopsy to find this file.

<img width="860" height="163" alt="image" src="https://github.com/user-attachments/assets/adefe867-33ed-4522-8309-a36f6ef64adb" />

Can you spot any irregularities? Timestomping is a crafty technique where adversaries modify a file's timestamps to blend in with surrounding files, making detection challenging for forensic tools and investigators. By accessing the file's metadata (File Metadata tab), we can pinpoint the MFT (Master File Table) attributes, which will reveal the genuine modification date.

Notably, there's a discrepancy when contrasting the `$FILE_NAME` MFT Modified value with the `$STANDARD_INFORMATION` File Modified value.

The `$STANDARD_INFORMATION` File Modified timestamp is what a user typically encounters when viewing file properties. This could lead someone to believe that the file has been present for a while and might be unrelated to any recent activity. However, `$FILE_NAME` MFT Modified holds the authentic timestamp, revealing the file's actual history.

<img width="664" height="576" alt="image" src="https://github.com/user-attachments/assets/a4a13f23-ba9f-4114-89cb-a506bfb27bf7" />

### Analyzing SRUM Data with Autopsy

Reflecting on our findings, we recall that the malicious executable had an open handle directed at the Desktop folder. Through Autopsy we notice a file named `users.db`. Given the circumstances, it's plausible that the attacker intended to siphon this data from the system.

<img width="472" height="715" alt="image" src="https://github.com/user-attachments/assets/7da85987-ab8b-49ff-8832-44fd32c1bd59" />

<img width="1494" height="282" alt="image" src="https://github.com/user-attachments/assets/81ac87da-ed0d-4629-9d35-8318b0ee9eeb" />

To validate our hypothesis, let's sift through Data Artifacts and access the Run Programs section. Our primary focus for network metadata analysis rests on `SRUDB.dat`.

<img width="507" height="276" alt="image" src="https://github.com/user-attachments/assets/a289fce5-a92a-4f14-9893-e602bfbb5d84" />

<img width="1356" height="345" alt="image" src="https://github.com/user-attachments/assets/a0b0408e-b5f1-4d65-8ed5-5beaca23c0c1" />

430526981 bytes may have been exfiltrated.

---

## Analyzing Rapid Triage Data - Windows Event Logs (Chainsaw)

In our pursuit of understanding the events that transpired, let's employ the Chainsaw utility (residing in `C:\Users\johndoe\Desktop\chainsaw`) to analyze the Windows Event Logs available at `C:\Users\johndoe\Desktop\kapefiles\auto\C%3A\Windows\System32\winevt\Logs` as follows. Our objective is to pinpoint key events that transpired during our incident timeline.

By piecing together the evidence, we can construct a comprehensive narrative of the attack, from its inception to its culmination.

<img width="743" height="529" alt="image" src="https://github.com/user-attachments/assets/8bdfbdb9-e212-4476-aa9d-cd35076aa5fd" />

The results will be available inside the `C:\Users\johndoe\Desktop\chainsaw\output_csv` directory of this section's target.

Upon examining `sigma.csv` (choose Fixed width in Separator Options), we observe the following alerts, among others related to the incident.

<img width="1605" height="615" alt="image" src="https://github.com/user-attachments/assets/1792824d-c35d-4461-993d-c61296b51d44" />

### Cobalt Strike Named Pipe

<img width="1562" height="246" alt="image" src="https://github.com/user-attachments/assets/fd62f046-36be-48e9-bb17-4277335983ee" />

<img width="1569" height="251" alt="image" src="https://github.com/user-attachments/assets/6af816d2-ea24-40a3-ad0a-b13d4466c368" />

<img width="1572" height="250" alt="image" src="https://github.com/user-attachments/assets/062b6559-8796-4f0c-8d83-f3cc6ff3fd6d" />

Cobalt Strike's named pipe functionality enables covert communication between adversaries and compromised systems, facilitating post-exploitation activities in a stealthy manner.

### UAC (User Account Control) Bypass/Privilege Escalation by Abusing fodhelper.exe

<img width="1616" height="616" alt="image" src="https://github.com/user-attachments/assets/6a597615-b3e1-4f6f-b75e-150883e40d2a" />

<img width="1582" height="618" alt="image" src="https://github.com/user-attachments/assets/468fe2ce-00f7-46bf-953a-40db8e02af05" />

https://pentestlab.blog/2017/06/07/uac-bypass-fodhelper/

### LSASS Access

<img width="1552" height="362" alt="image" src="https://github.com/user-attachments/assets/764f501c-ef33-4607-aaa6-db175cb690b7" />

### Windows PowerShell Execution

<img width="1623" height="264" alt="image" src="https://github.com/user-attachments/assets/ae54f876-8ee5-46d7-abd2-62725d1ce30f" />

Upon examining `account_tampering.csv`, we observe that a new user was created (Admin) and added to the Administrators group.

<img width="2175" height="610" alt="image" src="https://github.com/user-attachments/assets/42851f30-6714-43b3-bd4e-5961cedcfc9e" />

We can also identify new user creation through Autopsy as follows.

<img width="494" height="189" alt="image" src="https://github.com/user-attachments/assets/ef2661c9-59c9-42c0-bd68-f6ca0156e36e" />

<img width="1071" height="482" alt="image" src="https://github.com/user-attachments/assets/cecf33ca-e035-4089-a337-fc712b5937a8" />

---

## Analyzing Rapid Triage Data - Prefetch Files (PECmd)

Let's now dive into the system's execution history by analyzing the prefetch files (available at `C:\Users\johndoe\Desktop\kapefiles\auto\C%3A\Windows\Prefetch`) with `PECmd.exe`

<img width="741" height="841" alt="image" src="https://github.com/user-attachments/assets/2a83d2e4-fa27-4b0c-9a2f-bf14e871ebe2" />

<img width="927" height="757" alt="image" src="https://github.com/user-attachments/assets/585903cd-614a-47bd-ac8a-d3869693f8b0" />

---

## Analyzing Rapid Triage Data - USN Journal (usn.py)

Within the USN journal (available at `C:\Users\johndoe\Desktop\kapefiles\ntfs\%5C%5C.%5CC%3A:$Extend:$UsnJrnl%3A$J`), we can identify all files that were either created or deleted during the incident.

<img width="674" height="96" alt="image" src="https://github.com/user-attachments/assets/a107bef5-cf52-49c3-b0f8-31ac686444eb" />

Suspicious activities took place approximately between 2023-08-10 09:00:00 and 2023-08-10 10:00:00.

To view the CSV using PowerShell in alignment with our timeline, we can execute:

<img width="733" height="858" alt="image" src="https://github.com/user-attachments/assets/543dd55a-f727-4264-a3fc-35855055be44" />

<img width="734" height="804" alt="image" src="https://github.com/user-attachments/assets/9be8f759-217e-4979-b92d-6d9cab98038a" />

### Notable Activity

<img width="1177" height="260" alt="image" src="https://github.com/user-attachments/assets/a633fe9e-5159-4248-9385-dd352724543f" />

<img width="1179" height="1386" alt="image" src="https://github.com/user-attachments/assets/91390e4b-47bf-4386-b2e7-d4bb2d2c0a97" />

If we look carefully enough, we will notice that `flag.txt` was deleted.

---

## Analyzing Rapid Triage Data - MFT/pagefile.sys (MFTECmd/Autopsy)

We can leverage MFT in an attempt to recover `flag.txt`. Unfortunately, the affected machine's MFT table is not available.

For completeness' sake, let's work on another system's MFT table (available at `C:\Users\johndoe\Desktop\files\mft_data`) where `flag.txt` was also deleted.

Our initial step involves running MFTEcmd to parse the `$MFT` file, followed by searching for `flag.txt` within the report.

<img width="732" height="431" alt="image" src="https://github.com/user-attachments/assets/29a81d64-7653-4bcf-9aea-605da3264fb7" />

<img width="742" height="198" alt="image" src="https://github.com/user-attachments/assets/75f2d93a-7720-449d-a13c-8cec9b1aca06" />

The output provides the location of `flag.txt` on the system (`\Users\johndoe\Desktop\reports`).

Let's now access the MFT file (`C:\Users\johndoe\Desktop\files\mft_data`) using MFT Explorer (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6\MFTExplorer`)

On the Desktop, within the reports folder, we discover `flag.txt` marked with the Is deleted attribute.

<img width="1257" height="752" alt="image" src="https://github.com/user-attachments/assets/3723efd9-9af3-439d-8e59-919008fbe5ab" />

When files are deleted from an NTFS file system volume, their MFT entries are marked as free and may be reused, but the data may remain on the disk until overwritten. That’s why recovery isn't always possible.

In the case of the compromised system the file was overwritten (that's why we used the MFT table of another system for the recovery exercise), but portions of its content were preserved in `pagefile.sys`.

`pagefile.sys` is a designated system file in Windows that supplements your computer's RAM. When RAM nears its capacity, the system offloads less critical data, like certain files and applications, to the pagefile.

With knowledge of the file's partial content, we can scour the disk and retrieve our flag from `pagefile.sys` through Autopsy.

<img width="1316" height="94" alt="image" src="https://github.com/user-attachments/assets/4bccb643-7a59-4490-9da6-76f4f0de4ad1" />

<img width="687" height="379" alt="image" src="https://github.com/user-attachments/assets/b75f2668-cb88-4294-b6e8-3be94c369197" />

---

## Constructing an Execution Timeline

Given that the incident occurred between 09:13 and 09:30, we can use Autopsy to map out the attacker's actions chronologically.

Behind the scenes, Autopsy employs Plaso.

Let's make the following selections:

**Limit event types to:**
- Web Activity: All
- Other: All

**Set Display Times in:** GMT / UTC

**Start:** Aug 10, 2023 9:13:00 AM

**End:** Aug 10, 2023 9:30:00 AM

<img width="1891" height="1403" alt="image" src="https://github.com/user-attachments/assets/2508fe8b-915c-4ba9-b0b1-9b7c8994d982" />

This will allow us to generate a timeline detailing the actions undertaken by the malicious actor.

<img width="1468" height="834" alt="image" src="https://github.com/user-attachments/assets/73487b6b-8322-4a57-aaac-6f62cb096cbc" />

**List timeline view:**

<img width="1475" height="793" alt="image" src="https://github.com/user-attachments/assets/d7c94ac7-8852-4d3e-91eb-f4325473b3a2" />

By applying specific filters, we can pinpoint the files accessed or established during this particular window.

---

## The Actual Attack Timeline

Here are the real actions taken by the attacker (i.e. not identified through digital forensics). Based on what you've learned up to this point, attempt to recognize and pinpoint any of these actions that remain undetected in this section.

<img width="1180" height="1276" alt="image" src="https://github.com/user-attachments/assets/1f84d712-b4f8-41c9-9e24-3ad7fe45cd6d" />

<img width="1179" height="448" alt="image" src="https://github.com/user-attachments/assets/de9de494-c2b8-4c49-a401-8a103b285d03" />

---

## 🔑 Key Takeaway

This scenario demonstrates how combining memory analysis, disk forensics, and rapid triage artifacts can reconstruct a Windows compromise. Volatility helps identify suspicious processes, injected code, DLLs, handles, and network connections, while Autopsy and forensic tools reveal malicious downloads, persistence mechanisms, timestomping, execution artifacts, deleted files, and possible data exfiltration. Correlating these findings with Windows Event Logs, the USN Journal, and a Plaso-generated timeline helps build a chronological account of the attack.
