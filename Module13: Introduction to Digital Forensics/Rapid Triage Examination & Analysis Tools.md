<img width="1626" height="540" alt="image" src="https://github.com/user-attachments/assets/10924b7a-7c54-417e-9aef-02c227f86dd5" />## Rapid Triage Analysis Tools

When it comes to Rapid Triage analysis, the right external tools are essential for thorough examination and analysis.

Eric Zimmerman has curated a suite of indispensable tools tailored for this very purpose. These tools are meticulously designed to aid forensic analysts in their quest to extract vital information from digital devices and artifacts.

For a comprehensive list of these tools, check out: https://ericzimmerman.github.io/#!index.md

To streamline the download process, we can visit the official website and select either the .net 4 or .net 6 link. This action will initiate the download of all the tools in a compressed format.

<img width="1226" height="681" alt="image" src="https://github.com/user-attachments/assets/ac1d5c82-caf0-4046-be66-f62d5e1fe3e1" />

We can also leverage the provided PowerShell script, as outlined in step 2 of the screenshot above, to download all the tools.

<img width="737" height="859" alt="image" src="https://github.com/user-attachments/assets/f254cd7b-9126-4a58-8166-382e79bee65c" />

While we'll be utilizing a subset of these tools to analyze the KAPE output data, it's prudent for us to familiarize ourselves with the entire toolkit. Understanding the full capabilities of each tool can significantly enhance our investigative prowess.

In this section we will be working with certain tools and evidence that reside in the following directories of this section's target.

- **Evidence location:** `C:\Users\johndoe\Desktop\forensic_data`
- **KAPE's output location:** `C:\Users\johndoe\Desktop\forensic_data\kape_output`
- **Eric Zimmerman's tools location:** `C:\Users\johndoe\Desktop\Get-ZimmermanTools`
- **Active@ Disk Editor's location:** `C:\Program Files\LSoft Technologies\Active@ Disk Editor`
- **EQL's location:** `C:\Users\johndoe\Desktop\eqllib-master`
- **RegRipper's location:** `C:\Users\johndoe\Desktop\RegRipper3.0-master`

## MAC(b) Times in NTFS

The term MAC(b) times denotes a series of timestamps linked to files or objects. These timestamps are pivotal as they shed light on the chronology of events or actions on a file system. The acronym MAC(b) is an abbreviation for Modified, Accessed, Changed, and (b) Birth times. The inclusion of b signifies the Birth timestamp, which isn't universally present across all file systems or easily accessible via standard Windows API functions. Let's delve deeper into the nuances of MACB timestamps.

- **Modified Time (M):** This timestamp captures the last instance when the content within the file underwent modifications. Any alterations to the file's data, such as content edits, trigger an update to this timestamp.
- **Accessed Time (A):** This timestamp reflects the last occasion when the file was accessed or read, updating whenever the file is opened or otherwise engaged.
- **Changed [Change in MFT Record ] (C):** This timestamp signifies changes to the MFT record. It captures the moment when the file was initially created. However, it's worth noting that certain file systems, like NTFS, might update this timestamp if the file undergoes movement or copying.
- **Birth Time (b):** Often referred to as the Birth or Born timestamp, this represents the precise moment when the file or object was instantiated on the file system. Its significance in forensic investigations cannot be overstated, especially when determining a file's original creation time.

### General Rules for Timestamps in the Windows NTFS File System

The table below delineates the general rules governing how various file operations influence the timestamps within the Windows NTFS (New Technology File System).

| Operation | Modified | Accessed | Birth (Created) |
|---|---|---|---|
| File Create | Yes | Yes | Yes |
| File Modify | Yes | No | No |
| File Copy | No (Inherited) | Yes | Yes |
| File Access | No | No* | No |

**File Create:**

- **Modified Timestamp (M):** The Modified timestamp is updated to reflect the time of file creation.
- **Accessed Timestamp (A):** The Accessed timestamp is updated to reflect that the file was accessed at the time of creation.
- **Birth (Created) Timestamp (b):** The Birth timestamp is set to the time of file creation.

**File Modify:**

- **Modified Timestamp (M):** The Modified timestamp is updated to reflect the time when the file's content or attributes were last modified.
- **Accessed Timestamp (A):** The Accessed timestamp is not updated when the file is modified.
- **Birth (Created) Timestamp (b):** The Birth timestamp is not updated when the file is modified.

**File Copy:**

- **Modified Timestamp (M):** The Modified timestamp is typically not updated when a file is copied. It usually inherits the timestamp from the source file.
- **Accessed Timestamp (A):** The Accessed timestamp is updated to reflect that the file was accessed at the time of copying.
- **Birth (Created) Timestamp (b):** The Birth timestamp is updated to the time of copying, indicating when the copy was created.

**File Access:**

- **Modified Timestamp (M):** The Modified timestamp is not updated when the file is accessed.
- **Accessed Timestamp (A):** The Accessed timestamp is updated to reflect the time of access.
- **Birth (Created) Timestamp (b):** The Birth timestamp is not updated when the file is accessed.

All these timestamps reside in the `$MFT` file, located at the root of the system drive. While the `$MFT` file will be covered in greater depth later, our current focus remains on understanding these timestamps.

These timestamps are housed within the `$MFT` across two distinct attributes:

- `$STANDARD_INFORMATION`
- `$FILE_NAME`

The timestamps visible in the Windows file explorer are derived from the `$STANDARD_INFORMATION` attribute.

## Timestomping Investigation

Identifying instances of timestamp manipulation, commonly termed as timestomping (T1070.006), presents a formidable challenge in digital forensics. Timestomping entails the alteration of file timestamps to obfuscate the sequence of file activities. This tactic is frequently employed by various tools, as illustrated in the MITRE ATT&CK's timestomp technique.

<img width="1400" height="489" alt="image" src="https://github.com/user-attachments/assets/0b02414a-3444-44e9-871f-1d1d4db007ef" />

When adversaries manipulate file creation times or deploy tools for such purposes, the timestamp displayed in the file explorer undergoes modification.

For instance, if we load `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$MFT` into MFT Explorer (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6\MFTExplorer`) we will notice that the creation time of the file `ChangedFileTime.txt` has been tampered with, displaying 03-01-2022 in the file explorer, which deviates from the actual creation time.

**Note:** MFT Explorer will take 15-25 minutes to load the file.

<img width="1825" height="957" alt="image" src="https://github.com/user-attachments/assets/119ae31b-1484-4d07-a436-3d1e9c6dc749" />

However, given our knowledge that the timestamps in the file explorer originate from the `$STANDARD_INFORMATION` attribute, we can cross-verify this data with the timestamps from the `$FILE_NAME` attribute through MFTEcmd (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6`) as follows.

<img width="546" height="817" alt="image" src="https://github.com/user-attachments/assets/230012b8-5649-4911-acf0-bfa8175de6a0" />

<img width="1097" height="179" alt="image" src="https://github.com/user-attachments/assets/05a076e9-e4ee-40e9-a9c8-8b7c18f986b5" />

<img width="1059" height="300" alt="image" src="https://github.com/user-attachments/assets/69932336-92e7-4b09-a9a1-2264fd45b00c" />


In standard Windows file systems like NTFS, regular users typically lack the permissions to directly modify the timestamps of filenames in `$FILE_NAME`. Such modifications are exclusively within the purview of the system kernel.

To kickstart our exploration, let's first acquaint ourselves with filesystem-based artifacts. We'll commence with the `$MFT` file, nestled in the root directory of the KAPE output.

## MFT File

The `$MFT` file, commonly referred to as the Master File Table, is an integral part of the NTFS (New Technology File System) used by contemporary Windows operating systems. This file is instrumental in organizing and cataloging files and directories on an NTFS volume. Each file and directory on such a volume has a corresponding entry in the Master File Table. Think of the MFT as a comprehensive database, meticulously documenting metadata and structural details about every file and directory.

For those in the realm of digital forensics, the `$MFT` is a treasure trove of information. It offers a granular record of file and directory activities on the system, encompassing actions like file creation, modification, deletion, and access. By leveraging the `$MFT`, forensic analysts can piece together a detailed timeline of system events and user interactions.

**Note:** A standout feature of the MFT is its ability to retain metadata about files and directories, even post their deletion from the filesystem. This trait elevates the MFT's significance in forensic analysis and data recovery.

The MFT is strategically positioned at the root of the system drive.

We've already extracted the MFT while showcasing KAPE's capabilities and saved it at `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$MFT`.

Let's navigate to the `$MFT` file within the KAPE Output directory above and load it in MFT Explorer. This tool, one of Eric Zimmerman's masterpieces, empowers us to inspect and analyze the metadata nestled in the MFT. This encompasses a wealth of information about files and directories, from filenames and timestamps (created, modified, accessed) to file sizes, permissions, and attributes. A standout feature of the MFT Explorer is its intuitive interface, presenting file records in a graphical hierarchy reminiscent of the familiar Windows Explorer.

<img width="1463" height="612" alt="image" src="https://github.com/user-attachments/assets/4f0b8471-f1e3-480c-8fc4-84fa6b4f3f70" />

**Note:** It's worth noting that MFT records, once created, aren't discarded. Instead, as new files and directories emerge, new records are added to the MFT. Records corresponding to deleted files are flagged as "free" and stand ready for reuse.

### Structure of MFT File Record

Every file or directory on an NTFS volume is symbolized by a record in the MFT. These records adhere to a structured format, brimming with attributes and details about the associated file or directory. Grasping the MFT's structure is pivotal for tasks like forensic analysis, system management, and data recovery in Windows ecosystems. It equips forensic experts to pinpoint which attributes are brimming with intriguing insights.

<img width="2121" height="996" alt="image" src="https://github.com/user-attachments/assets/1b345008-dc3f-439c-a3eb-980a2ef401c3" />

Here's a snapshot of the components:

- **File Record Header:** Contains metadata about the file record itself. Includes fields like signature, sequence number, and other administrative data.
- **Standard Information Attribute Header:** Stores standard file metadata such as timestamps, file attributes, and security identifiers.
- **File Name Attribute Header:** Contains information about the filename, including its length, namespace, and Unicode characters.
- **Data Attribute Header:** Describes the file data attribute, which can be either resident (stored within the MFT record) or non-resident (stored in external clusters).
- **File Data (File content):** This section holds the actual file data, which can be the file's content or references to non-resident data clusters. For small files (less than 512 bytes), the data might be stored within the MFT record (resident). For larger files, it references non-resident data clusters on the disk. We'll see an example of this later on.
- **Additional Attributes (optional):** NTFS supports various additional attributes, such as security descriptors (SD), object IDs (OID), volume name (VOLNAME), index information, and more.

These attributes can vary depending on the file's characteristics. We can see the common type of information which is stored inside these header and attributes in the image below.

<img width="2098" height="1108" alt="image" src="https://github.com/user-attachments/assets/20187c79-b24c-4584-be2f-9faf8ecfe6ae" />

### File Record Header

Contains metadata about the file record itself. Includes fields like signature, sequence number, and other administrative data.

<img width="1226" height="485" alt="image" src="https://github.com/user-attachments/assets/3def20cc-e82a-4329-bee1-d6b5db223ee8" />

The file record begins with a header that contains metadata about the file record itself. This header typically includes the following information:

- **Signature:** A four-byte signature, usually "FILE" or "BAAD," indicating whether the record is in use or has been deallocated.
- **Offset to Update Sequence Array:** An offset to the Update Sequence Array (USA) that helps maintain the integrity of the record during updates.
- **Size of Update Sequence Array:** The size of the Update Sequence Array in words.
- **Log File Sequence Number:** A number that identifies the last update to the file record.
- **Sequence Number:** A number identifying the file record. The MFT records are numbered sequentially, starting from 0.
- **Hard Link Count:** The number of hard links to the file. This indicates how many directory entries point to this file record.
- **Offset to First Attribute:** An offset to the first attribute in the file record.

When we sift through the MFT file using MFTECmd and extract details about a record, the information from the file record is presented as depicted in the subsequent screenshot.

<img width="738" height="776" alt="image" src="https://github.com/user-attachments/assets/8128fadd-f0c5-45a5-afab-14e3523fd269" />

<img width="618" height="780" alt="image" src="https://github.com/user-attachments/assets/42ae37cc-4b8d-4d76-8035-271553d781b6" />


### MFT Attribute Types

Each attribute signifies some entry information, identified by type.

| Type | Attribute | Description |
|---|---|---|
| 0x10 (16) | `$STANDARD_INFORMATION` | General information - flags, MAC times, owner, and security id. |
| 0x20 (32) | `$ATTRIBUTE_LIST` | Pointers to other attributes and a list of nonresident attributes. |
| 0x30 (48) | `$FILE_NAME` | File name - (Unicode) and outdated MAC times |
| 0x40 (64) | `$VOLUME_VERSION` | Volume information - NTFS v1.2 only and Windows NT, no longer used |
| 0x40 (64) | `$OBJECT_ID` | 16B unique identifier - for file or directory (NTFS 3.0+; Windows 2000+) |
| 0x50 (80) | `$SECURITY_DESCRIPTOR` | File's access control list and security properties |
| 0x60 (96) | `$VOLUME_NAME` | Volume name |
| 0x70 (112) | `$VOLUME_INFORMATION` | File system version and other information |
| 0x80 (128) | `$DATA` | File contents |
| 0x90 (144) | `$INDEX_ROOT` | Root node of an index tree |
| 0xA0 (160) | `$INDEX_ALLOCATION` | Nodes of an index tree - with a root in `$INDEX_ROOT` |
| 0xB0 (176) | `$BITMAP` | Bitmap - for the `$MFT` file and for indexes (directories) |
| 0xC0 (192) | `$SYMBOLIC_LINK` | Soft link information - (NTFS v1.2 only and Windows NT) |
| 0xC0 (192) | `$REPARSE_POINT` | Data about a reparse point - used for a soft link (NTFS 3.0+; Windows 2000+) |
| 0xD0 (208) | `$EA_INFORMATION` | Used for backward compatibility with OS/2 applications (HPFS) |
| 0xE0 (224) | `$EA` | Used for backward compatibility with OS/2 applications (HPFS) |
| 0x100 (256) | `$LOGGED_UTILITY_STREAM` | Keys and other information about encrypted attributes (NTFS 3.0+; Windows 2000+) |

To demystify the structure of an NTFS MFT file record, we're harnessing the capabilities of Active@ Disk Editor. This potent, freeware disk editing tool is available at `C:\Program Files\LSoft Technologies\Active@ Disk Editor` and facilitates the viewing and modification of raw disk data, including the Master File Table of an NTFS system. The same insights can be gleaned from other MFT parsing tools, such as MFT Explorer.

We can have a closer look by opening `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$MFT` on Active@ Disk Editor and then pressing Inspect File Record.

<img width="3075" height="1773" alt="image" src="https://github.com/user-attachments/assets/5ebc4726-32ed-4a77-8854-14337d70c1fd" />

<img width="3075" height="1773" alt="image" src="https://github.com/user-attachments/assets/684ef510-8927-4133-bced-e88511a35434" />

In Disk Editor, we're privy to the raw data of MFT entries. This includes a hexadecimal representation of the MFT record, complete with its header and attributes.

### Non-Resident Flag

<img width="2017" height="757" alt="image" src="https://github.com/user-attachments/assets/57854293-826f-4888-ab07-34327761db2a" />

When parsing the entry in MFTECmd, this is how the non-resident data header appears.

<img width="1726" height="999" alt="image" src="https://github.com/user-attachments/assets/7e41abd0-d9cc-47e3-b548-2e59cfcd7c64" />

### Resident Flag

<img width="2030" height="761" alt="image" src="https://github.com/user-attachments/assets/06198f0f-7590-4b3f-af14-a4fafab6d6e7" />

When parsing the entry in MFTECmd, this is how the resident data header appears.


### Zone.Identifier data in MFT File Record

The Zone.Identifier is a specialized file metadata attribute in the Windows OS, signifying the security zone from which a file was sourced. It's an integral part of the Windows Attachment Execution Service (AES) and is instrumental in determining how Windows processes files procured from the internet or other potentially untrusted origins.

When a file is fetched from the internet, Windows assigns it a Zone Identifier (ZoneId). This ZoneId, embedded in the file's metadata, signifies the source or security zone of the file's origin. For instance, internet-sourced files typically bear a ZoneId of 3, denoting the Internet Zone.

For instance, we downloaded various tools inside the `C:\Users\johndoe\Downloads` directory of this section's target. Post-download, a ZoneID replete with the Zone.Identifier (i.e., the source URL) has been assigned to them.

<img width="742" height="810" alt="image" src="https://github.com/user-attachments/assets/b0479a32-e286-4697-bea2-089eac67b581" />

<img width="702" height="523" alt="image" src="https://github.com/user-attachments/assets/aea22c97-03f5-4d37-9071-777229d6c234" />

To unveil the content of a Zone.Identifier for a file, the following command can be executed in PowerShell.

<img width="737" height="623" alt="image" src="https://github.com/user-attachments/assets/fd8787af-edd0-436d-a3b6-88655998a8b9" />

One of the security mechanisms, known as the Mark of the Web (MotW), hinges on the Zone Identifier. Here, the MotW marker differentiates files sourced from the internet or other potentially dubious sources from those originating from trusted or local contexts. It's frequently employed to bolster the security of applications like Microsoft Word. When an app, say Microsoft Word, opens a file bearing a MotW, it can institute specific security measures based on the MotW's presence. For instance, a Word document with a MotW might be launched in Protected View, a restricted mode that isolates the document from the broader system, mitigating potential security threats.

While its primary function is to bolster security for files downloaded from the web, forensic analysts can harness it for investigative pursuits. By scrutinizing this attribute, they can ascertain the file's download method. See an example below.

<img width="3069" height="1711" alt="image" src="https://github.com/user-attachments/assets/635d7f27-bffc-4676-b145-e6227552691b" />

## Analyzing with Timeline Explorer

Timeline Explorer is another digital forensic tool developed by Eric Zimmerman which is used to assist forensic analysts and investigators in creating and analyzing timeline artifacts from various sources. Timeline artifacts provide a chronological view of system events and activities, making it easier to reconstruct a sequence of events during an investigation. We can filter timeline data based on specific criteria, such as date and time ranges, event types, keywords, and more. This feature helps focus the investigation on relevant information.

This arrangement of different events following one after another in time is really useful to create a story or timeline about what happened before and after specific events. This sequencing of events helps establish a timeline of activities on a system.

Loading a converted CSV file into Timeline Explorer is a straightforward process. Timeline Explorer is designed to work with timeline data, including CSV files that contain timestamped events or activities. To load the event data csv file into the Timeline Explorer, we can launch Timeline Explorer, and simply drag and drop from its location (e.g., our KAPE analysis directory) onto the Timeline Explorer window.

Once ingested, Timeline Explorer will process and display the data. The duration of this process hinges on the file's size.

<img width="1298" height="603" alt="image" src="https://github.com/user-attachments/assets/294ef8a3-1397-415a-bd34-ecdf1d07b8c2" />

We will see the timeline populated with the events from the CSV file in chronological order. With the timeline data now loaded, we can explore and analyze the events using the various features provided by Timeline Explorer. We can zoom in on specific time ranges, filter events, search for keywords, and correlate related activities.

<img width="1711" height="691" alt="image" src="https://github.com/user-attachments/assets/2a875ce5-9bc2-40f7-8569-7c4dd4d95ca6" />

We will provide multiple examples of using Timeline Explorer in this section.

## USN Journal

USN, or Update Sequence Number, is a vital component of the NTFS file system in Windows. The USN Journal is essentially a change journal feature that meticulously logs alterations to files and directories on an NTFS volume.

For those in digital forensics, the USN Journal is a goldmine. It enables us to monitor operations such as File Creation, Rename, Deletion, and Data Overwrite.

In the Windows environment, the USN Journal file is designated as `$J`. The KAPE Output directory houses the collected USN Journal in the following directory: `<KAPE_output_folder>\<Drive>\$Extend`

Here is how it looks in our KAPE's output (`C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$Extend`)

<img width="1284" height="269" alt="image" src="https://github.com/user-attachments/assets/bd50b2d5-7f7f-4f6d-90ea-4a99653efc4a" />

### Analyzing the USN Journal Using MFTECmd

We previously utilized MFTECmd, one of Eric Zimmerman's tools, to parse the MFT file. While its primary focus is the MFT, MFTECmd can also be instrumental in analyzing the USN Journal. This is because entries in the USN Journal often allude to modifications to files and directories that are documented in the MFT. Hence, we'll employ this tool to dissect the USN Journal.

To facilitate the analysis of the USN Journal using MFTECmd, execute a command akin to the one below:

<img width="713" height="505" alt="image" src="https://github.com/user-attachments/assets/8a68a5ec-2a54-4331-a16f-6a249b7f3254" />

The resultant output file is saved as `MFT-J.csv` inside the `C:\Users\johndoe\Desktop\forensic_data\mft_analysis` directory. Let's import it into Timeline Explorer (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6\TimelineExplorer`).

**Note:** Please remove the filter on the Entry Number to see the whole picture.

<img width="1920" height="841" alt="image" src="https://github.com/user-attachments/assets/67e62a4d-c065-4695-95c8-14b5c7107f83" />

Upon inspection, we can discern a chronologically ordered timeline of events. Notably, the entry for `uninstall.exe` is evident.

By applying a filter on the Entry Number 93866, which corresponds to the Entry ID for `uninstall.exe`, we can glean the nature of modifications executed on this specific file.

<img width="1402" height="530" alt="image" src="https://github.com/user-attachments/assets/ead5cd38-09c6-4dfd-8937-1f081c7dad5f" />

The file extension, `.crdownload`, is indicative of a partially downloaded file. This type of file is typically generated when downloading content via browsers like Microsoft Edge, Google Chrome, or Chromium. This revelation is intriguing. If the file was downloaded via a browser, it's plausible that the Zone.Identifier could unveil the source IP/domain of its origin.

To investigate this assumption we should:

1. Create a CSV file for `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$MFT` using MFTECmd as we did for `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\$Extend\$J`.
2. Import the `$MFT`-related CSV into Timeline Explorer.
3. Apply a filter on the entry Number 93866.

<img width="735" height="477" alt="image" src="https://github.com/user-attachments/assets/016ff7c7-f057-4a0d-b45b-9e03b0df04dc" />

<img width="1271" height="580" alt="image" src="https://github.com/user-attachments/assets/d0f5f55e-c2de-445f-8bed-2618c0c881e8" />

## Windows Event Logs Investigation

Probing into Windows Event Logs is paramount in digital forensics and incident response. These logs are repositories of invaluable data, chronicling system activities, user behaviors, and security incidents on a Windows machine. When KAPE is executed, it duplicates the original event logs, ensuring their pristine state is preserved as evidence. The KAPE Output directory houses these event logs in the following directory: `<KAPE_output_folder>\Windows\System32\winevt\logs`

<img width="1777" height="766" alt="image" src="https://github.com/user-attachments/assets/dc6d24c7-8339-4c86-a12c-c2c6b0dd9f3d" />

This directory is populated with `.evtx` files, encapsulating a myriad of windows event logs, including but not limited to Security, Application, System, and Sysmon (if activated).

Our mission is to sift through these event logs, on the hunt for any anomalies, patterns, or indicators of compromise (IOCs). As forensic sleuths, we should be vigilant, paying heed to event IDs, timestamps, source IPs, usernames, and other pertinent log details. A plethora of forensic utilities and scripts, such as log parsing tools and SIEM systems, can bolster our analysis. It's imperative to identify the tactics, techniques, and procedures (TTPs) evident in any dubious activity. This might entail delving into known attack patterns and malware signatures. Another crucial step is to correlate events from diverse log sources, crafting a comprehensive timeline of events. This holistic view aids in piecing together the sequence of events.

The analysis of Windows Event Logs has been addressed in the modules titled Windows Event Logs & Finding Evil and YARA & Sigma for SOC Analysts.

### Windows Event Logs Parsing Using EvtxECmd (EZ-Tool)

EvtxECmd (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6\EvtxeCmd`) is another brainchild of Eric Zimmerman, tailored for Windows Event Log files (EVTX files). With this tool at our disposal, we can extract specific event logs or a range of events from an EVTX file, converting them into more digestible formats like JSON, XML, or CSV.

Let's initiate the help menu of EvtxECmd to familiarize ourselves with the various options. The command to access the help section is as follows.

<img width="737" height="777" alt="image" src="https://github.com/user-attachments/assets/98a5996a-1c62-4645-8f33-4dd220d43deb" />

<img width="725" height="490" alt="image" src="https://github.com/user-attachments/assets/ac863be8-611d-43de-91b4-874f4c64f0b7" />

<img width="1724" height="826" alt="image" src="https://github.com/user-attachments/assets/8eea0a36-75ae-4b74-b744-51ac0e4f8b32" />

### Maps in EvtxECmd

Maps in EvtxECmd are pivotal. They metamorphose customized data into standardized fields in the CSV (and JSON) data. This granularity and precision are indispensable in forensic investigations, enabling analysts to interpret and extract salient information from Windows Event Logs with finesse.

**Standardized fields in maps:**

- **UserName:** Contains information about user and/or domain found in various event logs
- **ExecutableInfo:** Contains information about process command line, scheduled tasks etc.
- **PayloadData1,2,3,4,5,6:** Additional fields to extract and put contextual data from event logs
- **RemoteHost:** Contains information about IP address

EvtxECmd plays a significant role in:

- Converting the unique part of an event, known as EventData, into a more standardized and human-readable format.
- Ensuring that the map files are tailored to specific event logs, such as Security, Application, or custom logs, to handle differences in event structures and data.
- Using a unique identifier, the Channel element, to specify which event log a particular map file is designed for, preventing confusion when event IDs are reused across different logs.

To ensure the most recent maps are in place before converting the EVTX files to CSV/JSON, employ the command below.

<img width="732" height="778" alt="image" src="https://github.com/user-attachments/assets/b249507a-4514-4a7e-aeb3-c7e5710d6999" />

With the latest maps integrated, we're equipped to infuse contextual information into distinct fields, streamlining the log analysis process. Now, it's time to transmute the logs into a format that's more palatable.

To render the EVTX files more accessible, we can employ EvtxECmd to seamlessly convert event log files into user-friendly formats like JSON or CSV.

For instance, the command below facilitates the conversion of the `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\winevt\logs\Microsoft-Windows-Sysmon%4Operational.evtx` file to a CSV file:

<img width="548" height="751" alt="image" src="https://github.com/user-attachments/assets/4e3e6e64-02c6-4ee3-9315-3b09b0e32a5a" />

After importing the resultant CSV into Timeline Explorer, we should see the below.

<img width="1920" height="787" alt="image" src="https://github.com/user-attachments/assets/10ead100-6f5e-4fd8-ab87-74f9778c1acf" />

**Executable Information:**

<img width="1633" height="380" alt="image" src="https://github.com/user-attachments/assets/75c49262-20e0-4fbe-b4c7-630a9c8d9683" />

## Investigating Windows Event Logs with EQL

Endgame's Event Query Language (EQL) is an indispensable tool for sifting through event logs, pinpointing potential security threats, and uncovering suspicious activities on Windows systems. EQL offers a structured language that facilitates querying and correlating events across multiple log sources, including the Windows Event Logs.

Currently, the EQL module is compatible with Python versions 2.7 and 3.5+. If you have a supported Python version installed, execute the following command.

```cmd
C:\Users\johndoe>pip install eql
```

Should Python be properly configured and included in your PATH, eql should be accessible. To verify this, execute the command below.

```cmd
C:\Users\johndoe>eql --version
eql 0.9.18
```

Within EQL's repository (available at `C:\Users\johndoe\Desktop\eqllib-master`), there's a PowerShell module brimming with essential functions tailored for parsing Sysmon events from Windows Event Logs. This module resides in the utils directory of eqllib, and is named `scrape-events.ps1`.

From the EQL directory, initiate the scrape-events.ps1 module with the following command:

```powershell
PS C:\Users\johndoe\Desktop\eqllib-master\utils> import-module .\scrape-events.ps1
```

By doing so, we activate the `Get-EventProps` function, which is instrumental in parsing event properties from Sysmon logs. To transform, for example, `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\winevt\logs\Microsoft-Windows-Sysmon%4Operational.evtx` into a JSON format suitable for EQL queries, execute the command below.

<img width="735" height="140" alt="image" src="https://github.com/user-attachments/assets/3463ed12-45ac-4f87-ad9b-6b9340259b6f" />

This action will yield a JSON file, primed for EQL queries.

Let's now see how we could have identified user/group enumeration through an EQL query against the JSON file we created.

<img width="733" height="837" alt="image" src="https://github.com/user-attachments/assets/ccb21281-8ecd-4d46-8bcc-c55ce26d6bcb" />

<img width="706" height="68" alt="image" src="https://github.com/user-attachments/assets/7bd778b6-80fd-4371-8490-99e196f46bb1" />

<img width="1685" height="799" alt="image" src="https://github.com/user-attachments/assets/a6b2af84-2f92-454b-9531-4ef4fd84023e" />

## Windows Registry

A deep dive into the registry hives can furnish us with invaluable insights, such as the computer's name, Windows version, owner's name, and network configuration.

Registry-related files harvested from KAPE are typically housed in `<KAPE_output_folder>\Windows\System32\config`

<img width="1303" height="383" alt="image" src="https://github.com/user-attachments/assets/5079a65c-df01-4f6a-8b10-00a215de3059" />

Additionally, there are user-specific registry hives located within individual user directories, as exemplified in the following screenshot.

<img width="1166" height="503" alt="image" src="https://github.com/user-attachments/assets/89671b44-b91b-41c7-ba4d-f6c901485092" />

For a comprehensive analysis, we can employ Registry Explorer (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6\RegistryExplorer`) a GUI-based tool masterminded by Eric Zimmerman. This tool offers a streamlined interface to navigate and dissect the contents of Windows Registry hives. By simply dragging and dropping these files into Registry Explorer, the tool processes the data, presenting it within its GUI. The left panel displays the registry hives, while the right panel reveals their corresponding values.

In the screenshot below we have loaded the SYSTEM hive, that can be found inside the `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\config` directory of this section's target.

<img width="3303" height="1447" alt="image" src="https://github.com/user-attachments/assets/2eb4f3ff-4d2f-4460-af37-d8319144570b" />

Registry Explorer boasts a suite of features, including hive analysis, search capabilities, filtering options, timestamp viewing, and bookmarking. The bookmarking utility is particularly handy, allowing users to earmark pivotal locations or keys for subsequent reference.

In the screenshot below we have loaded the SOFTWARE hive, that can be found inside the `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\config` directory of this section's target. Notice the available bookmarks within Registry Explorer.

<img width="3297" height="1451" alt="image" src="https://github.com/user-attachments/assets/9d5b1a49-888d-4b98-84c7-306e52550c30" />

## RegRipper

Another potent tool in our arsenal is RegRipper (available at `C:\Users\johndoe\Desktop\RegRipper3.0-master`), a command-line utility adept at swiftly extracting information from the Registry.

To acquaint ourselves with RegRipper's functionalities, let's invoke the help section by executing `rip.exe` accompanied by the `-h` parameter.

<img width="733" height="694" alt="image" src="https://github.com/user-attachments/assets/32ddedef-40f7-4913-88b1-4dbbaa675dec" />

For a seamless experience with RegRipper, it's essential to familiarize ourselves with its plugins. To enumerate all available plugins and catalog them in a CSV file (e.g., `rip_plugins.csv`), use the command below.

```powershell
PS C:\Users\johndoe\Desktop\RegRipper3.0-master> .\rip.exe -l -c > rip_plugins.csv
```

This action compiles a comprehensive list of plugins, detailing the associated hives, and saves it as a CSV file.

The screenshot below elucidates the contents of this file, highlighting the plugin name, its corresponding registry hive, and a brief description.

<img width="1626" height="540" alt="image" src="https://github.com/user-attachments/assets/5da77375-c1eb-4349-a599-4078aa6011cf" />

To kick things off, let's execute the `compname` command on the SYSTEM hive (located at `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\config`), which retrieves the computer's name.

<img width="736" height="208" alt="image" src="https://github.com/user-attachments/assets/f26f3d02-5c1a-474b-9044-9d8028eb157c" />

Let's see some more examples against different hives.

### Timezone

<img width="720" height="347" alt="image" src="https://github.com/user-attachments/assets/dcacf240-2012-4c77-9803-e4b20db916ba" />

### Network Information

D<img width="614" height="765" alt="image" src="https://github.com/user-attachments/assets/c2676dc7-d563-419d-8fba-a8b556823ea2" />

The same information can be extracted using the `ips` plugin.

### Installer Execution

<img width="646" height="673" alt="image" src="https://github.com/user-attachments/assets/82587577-6fad-4c8e-ae64-7513b60c3466" />

### Recently Accessed Folders/Docs

<img width="633" height="541" alt="image" src="https://github.com/user-attachments/assets/ec763f99-c825-4a8e-9fd7-458586afde87" />

### Autostart - Run Key Entries

<img width="637" height="738" alt="image" src="https://github.com/user-attachments/assets/2ebf335c-c956-4534-a513-ed4fa2253bed" />

## Program Execution Artifacts

When we talk about execution artifacts in digital forensics, we're referring to the traces and evidence left behind on a computer system or device when a program runs. These little bits of information can clue us in on the activities and behaviors of software, users, and even those with malicious intent. If we want to piece together what went down on a computer, diving into these execution artifacts is a must.

You might stumble upon some well-known execution artifacts in these Windows components:

- Prefetch
- ShimCache
- Amcache
- BAM (Background Activity Moderator)

Let's dive deeper into each of these to get a better grasp on the kind of program execution details they capture.

### Investigation of Prefetch

Prefetch is a Windows operating system feature that helps optimize the loading of applications by preloading certain components and data. Prefetch files are created for every program that is executed on a Windows system, and this includes both installed applications and standalone executables. The naming convention of Prefetch files is indeed based on the original name of the executable file, followed by a hexadecimal value of the path where the executable file resides, and it ends with the `.pf` file extension.

In digital forensics, the Prefetch folder and associated files can provide valuable insights into the applications that have been executed on a Windows system. Forensic analysts can examine Prefetch files to determine which applications have been run, how often they were executed, and when they were last run.

In general, prefetch files are stored in the `C:\Windows\Prefetch\` directory.

Prefetch-related files harvested from KAPE are typically housed in `<KAPE_output_folder>\Windows\prefetch`.

<img width="922" height="320" alt="image" src="https://github.com/user-attachments/assets/bafbf221-8053-4cf8-b144-5857f345a344" />

Eric Zimmerman provides a tool for prefetch files: PECmd (available at `C:\Users\johndoe\Desktop\Get-ZimmermanTools\net6`).

Here's an example of how to launch PECmd's help menu from the EricZimmerman tools directory.

<img width="657" height="825" alt="image" src="https://github.com/user-attachments/assets/480da5ce-253d-4e45-b720-9685b6943be5" />

<img width="641" height="232" alt="image" src="https://github.com/user-attachments/assets/ec56c4ff-075f-43fc-a3be-a99630a96b70" />

<img width="1520" height="802" alt="image" src="https://github.com/user-attachments/assets/c533b778-227e-466a-9e6a-9d985cbb56df" />

PECmd will analyze the prefetch file (`.pf`) and display various information about the application execution. This generally includes details such as:

- First and last execution timestamps.
- Number of times the application has been executed.
- Volume and directory information.
- Application name and path.
- File information, such as file size and hash values.

Let's see by providing a path to a single prefetch file, for example the prefetch file related to `discord.exe` (i.e. `DISCORD.EXE-7191FAD6.pf` located at `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\prefetch`).

<img width="648" height="706" alt="image" src="https://github.com/user-attachments/assets/fdeabe81-2fcb-402f-b796-b9a2c434d972" />

<img width="651" height="569" alt="image" src="https://github.com/user-attachments/assets/2ae961bf-f6a6-4811-9513-a883d2859dcc" />

<img width="647" height="648" alt="image" src="https://github.com/user-attachments/assets/79eb31a2-9b33-47bf-9827-f3aa32afcaac" />

<img width="654" height="801" alt="image" src="https://github.com/user-attachments/assets/ef850188-4e3f-4c72-abbb-0123e298f14a" />

<img width="616" height="219" alt="image" src="https://github.com/user-attachments/assets/ea8dfbd9-512e-4fc1-a16a-363862a5df6c" />

Upon scrolling down the output, we can see the directories referenced by this executable.

<img width="1344" height="617" alt="image" src="https://github.com/user-attachments/assets/460e866a-46aa-4807-91f8-6e49f71d0439" />

Further scrolling down the output reveals the files referenced by this executable.

<img width="1331" height="614" alt="image" src="https://github.com/user-attachments/assets/a71cf380-62f6-4926-94c0-70c7421a28c9" />

#### Suspicious Activity in Referenced Files

We should also consider the directory where the application was executed from. If it was run from an unusual or unexpected location, it may be suspicious. For example the below screenshot shows some suspicious locations and files.

<img width="1472" height="491" alt="image" src="https://github.com/user-attachments/assets/c4b78b49-f487-4a44-ba81-52589274cf95" />

#### Convert Prefetch Files to CSV

For easier analysis, we can convert the prefetch data into CSV as follows.

<img width="648" height="812" alt="image" src="https://github.com/user-attachments/assets/dae01d85-2b36-4bf8-964d-ec2e633e591a" />

<img width="653" height="779" alt="image" src="https://github.com/user-attachments/assets/ab5e360c-9a60-4dff-ad72-c84d8e8d3a49" />

<img width="664" height="803" alt="image" src="https://github.com/user-attachments/assets/91335d0b-c38c-480d-835b-e00fa0df9092" />

<img width="654" height="292" alt="image" src="https://github.com/user-attachments/assets/bd3c78cc-bc05-41e3-a698-fe567496aff7" />

The destination directory contains the parsed output in CSV format.

<img width="999" height="189" alt="image" src="https://github.com/user-attachments/assets/0211a9a9-20ad-4d42-8c29-03cfc7cd0695" />

Now we can easily analyse the output in Timeline Explorer. Let's load both files.

<img width="1804" height="742" alt="image" src="https://github.com/user-attachments/assets/18763f8e-2459-4035-83ab-1c4c9f0b4e36" />

The second output file is the timeline file, which shows the executable details sorted by the run time.

<img width="1520" height="711" alt="image" src="https://github.com/user-attachments/assets/d15440d6-f5c5-4228-8fb1-b528911451a9" />

### Investigation of ShimCache (Application Compatibility Cache)

ShimCache (also known as AppCompatCache) is a Windows mechanism used by the Windows operating systems in order to identify application compatibility issues. This database records information about executed applications, and is stored in the Windows Registry. This information can be used by developers to track compatibility issues with executed programs.

In the AppCompatCache cache entries, we can see the information such as:

- Full file paths
- Timestamps
- Last modified time ($Standard_Information)
- Last updated time (Shimcache)
- Process execution flag
- Cache entry position

Forensic investigators can use this information to detect the execution of potentially malicious files.

The AppCompatCache key is located at the `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Control\Session Manager\AppCompatCache` registry location.

Let's load the SYSTEM registry hive (available at `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\System32\config`) in Registry Explorer and see what kind of information it contains. We can do that by opening Registry Explorer and dropping the registry hive files into it. Then we will need to go to bookmarks and select AppCompatCache. In the bottom right side, we should see the evidence of application execution as shown in the screenshot.

<img width="3301" height="1455" alt="image" src="https://github.com/user-attachments/assets/6b3c5ba4-a075-42e3-b598-9123ed0a7b0d" />

### Investigation of Amcache

AmCache refers to a Windows registry file which is used to store evidence related to program execution. It serves as a valuable resource for digital forensics and security investigations, helping analysts understand the history of application execution and detect signs of any suspicious execution.

The information that it contains include the execution path, first executed time, deleted time, and first installation. It also provides the file hash for the executables.

On Windows OS the AmCache hive is located at `C:\Windows\AppCompat\Programs\AmCache.hve`

AmCache-related files harvested from KAPE are typically housed in `<KAPE_output_folder>\Windows\AppCompat\Programs`.

Let's load `C:\Users\johndoe\Desktop\forensic_data\kape_output\D\Windows\AppCompat\Programs\AmCache.hve` in Registry Explorer to see what kind of information it contains.

<img width="3325" height="1451" alt="image" src="https://github.com/user-attachments/assets/c59c3743-c7a7-404d-9d08-54c9f8d6a5d2" />

Using Eric Zimmerman's AmcacheParser, we can parse and convert this file into a CSV, and analyze it in detail inside Timeline Explorer.

<img width="642" height="579" alt="image" src="https://github.com/user-attachments/assets/ccaf55ed-b9d3-4219-a415-05cf75749e4e" />

### Investigation of Windows BAM (Background Activity Moderator)

The Background Activity Moderator (BAM) is a component in the Windows operating system that tracks and logs the execution of certain types of background or scheduled tasks. BAM is actually a kernel device driver as shown in the below screenshot.

<img width="988" height="363" alt="image" src="https://github.com/user-attachments/assets/99956a7d-b7af-4a51-849d-111d02427f7c" />

It is primarily responsible for controlling the activity of background applications but it can help us in providing the evidence of program execution which it lists under the bam registry hive. The BAM key is located at the below registry location. `HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\bam\State\UserSettings\{USER-SID}`

Using Registry Explorer, we can browse this inside the SYSTEM hive to see the executable names. Registry explorer already has a bookmark for bam.

<img width="3300" height="1617" alt="image" src="https://github.com/user-attachments/assets/5bef9e19-555a-4163-b848-60bbe4c2ba68" />

We can also use RegRipper to get similar information through its `bam` plugin.

## Analyzing Captured API Call Data (.apmx64)

`.apmx64` files are generated by API Monitor, which records API call data. These files can be opened and analyzed within the tool itself. API Monitor is a software that captures and displays API calls initiated by applications and services. While its primary function is debugging and monitoring, its capability to capture API call data makes it handy for uncovering forensic artifacts. Let's proceed by loading `C:\Users\johndoe\Desktop\forensic_data\APMX64\discord.apmx64` into API Monitor (available at `C:\Program Files\rohitab.com\API Monitor`) and examining its contents for valuable information.

Launching the API Monitor will initiate certain necessary files.

<img width="707" height="334" alt="image" src="https://github.com/user-attachments/assets/e0bc196d-2042-4b51-85f1-05f265d26146" />

Upon opening the API Monitor application, let's head over to the File menu and choose Open From there, let's navigate to the location of the `.apmx64` file and select it.

<img width="1120" height="568" alt="image" src="https://github.com/user-attachments/assets/b2ff1bfe-2358-404e-96ec-593f4ee34494" />

After opening the file, a list of recorded API calls made by the monitored application will be displayed. Typically, this list contains details such as the API function name, parameters, return values, and timestamps. The screenshot below offers a comprehensive view of the API Monitor user interface and its various sections.

<img width="1683" height="820" alt="image" src="https://github.com/user-attachments/assets/7fab71e3-26c1-4c88-9abe-6e50f5492bc2" />

Clicking on the monitored processes to the left will display the recorded API call data for the chosen process in the summary view to the right. For illustration, consider selecting the `discord.exe` process. In the summary view, we will observe the API calls initiated by `discord.exe`.

<img width="1366" height="467" alt="image" src="https://github.com/user-attachments/assets/131c2783-6a09-426f-80d2-7a6c8d165413" />

A notable observation from the screenshot is the call to the `getenv` function. Here's the syntax of this function.

```c
char *getenv(
   const char *varname
);
```

This function retrieves the value of a specified environment variable. It requires a varname parameter, representing a valid environment variable name, and returns a pointer pointing to the table entry containing the respective environment variable's value.

API Monitor boasts a plethora of filtering and search capabilities. This allows us to hone in on specific API calls based on functions or time frames. By browsing through the summary or utilizing the filter and search functionalities, we can unearth intriguing details, such as API calls concerning file creation, process creation, registry alterations, and more.

## Registry Persistence via Run Keys

An oft-employed strategy by adversaries to maintain unauthorized access to a compromised system is inserting an entry into the run keys within the Windows Registry. Let's investigate if there's any reference to the `RegOpenKeyExA` function, which accesses the designated registry key. To perform this search, simply type RegOpenKey into the search box, usually situated atop the API Monitor window, and press Enter.

<img width="1335" height="495" alt="image" src="https://github.com/user-attachments/assets/eb228347-5d75-429a-9584-82c69f8249c5" />

From the displayed results, it's evident that the registry key `SOFTWARE\Microsoft\Windows\CurrentVersion\Run` corresponds to the Run registry key, which triggers the designated program upon every user login. Malicious entities often exploit this key to embed entries pointing to their backdoor, a task achievable via the registry API function `RegSetValueExA`.

To explore further, let's seek any mention of the `RegSetValueExA` function, which defines data and type for a specified value within a registry key. Engage the search box, type RegSet, and hit Enter.

<img width="1407" height="586" alt="image" src="https://github.com/user-attachments/assets/2b8c32da-51a3-46ce-99ba-debde67d8658" />

A notable observation is the `RegSetValueExA` invocation. Before diving deeper, let's familiarize ourselves with this function's documentation.

```c
LSTATUS RegSetValueExA(
  [in]           HKEY       hKey,
  [in, optional] LPCSTR     lpValueName,
                 DWORD      Reserved,
  [in]           DWORD      dwType,
  [in]           const BYTE *lpData,
  [in]           DWORD      cbData
);
```

- `hKey1` is a handle to the registry key where you want to set a registry value.
- `lpValueName` is a pointer to a null-terminated string that specifies the name of the registry value you want to set. In this case, it is named as DiscordUpdate.
- The `Reserved` parameter is reserved and must be zero.
- `dwType` specifies the data type of the registry value. It's likely an integer constant that represents the data type (e.g., REG_SZ for a string value).
- `(BYTE*)lpData` is a type cast that converts the _lpData_ variable to a pointer to a byte (BYTE*). This is done to ensure that the data pointed to by _lpData_ is treated as a byte array, which is the expected format for binary data in the Windows Registry. In our case, this is shown in the buffer view as `C:\Windows\Tasks\update.exe`.
- `cbData` is an integer that specifies the size, in bytes, of the data pointed to by _lpData_.

<img width="1500" height="309" alt="image" src="https://github.com/user-attachments/assets/a96706b0-64ae-4b1d-9d98-414602a6db2e" />

A critical takeaway from this API call is the `lpData` parameter, which reveals the backdoor's location, `C:\Windows\Tasks\update.exe`.

## Process Injection

To scrutinize process creation, let's search for the `CreateProcessA` function. Let's key in CreateProcess in the search box and press Enter.

<img width="1681" height="780" alt="image" src="https://github.com/user-attachments/assets/564af59d-36ce-4d86-a31b-ce739d64076c" />

Presented below is the syntax of the Windows API function, `CreateProcessA`.

```c
BOOL CreateProcessA(
  [in, optional]      LPCSTR                lpApplicationName,
  [in, out, optional] LPSTR                 lpCommandLine,
  [in, optional]      LPSECURITY_ATTRIBUTES lpProcessAttributes,
  [in, optional]      LPSECURITY_ATTRIBUTES lpThreadAttributes,
  [in]                BOOL                  bInheritHandles,
  [in]                DWORD                 dwCreationFlags,
  [in, optional]      LPVOID                lpEnvironment,
  [in, optional]      LPCSTR                lpCurrentDirectory,
  [in]                LPSTARTUPINFOA        lpStartupInfo,
  [out]               LPPROCESS_INFORMATION lpProcessInformation
);
```

An intriguing element within this API is the `lpCommandLine` parameter. It discloses the executed command line, which, in this context, is `C:\Windows\System32\comp.exe`. Notably, the `lpCommandLine` can be specified without delineating the complete executable path in the `lpApplicationName` value.

Another pivotal parameter worth noting is `dwCreationFlags`, set to `CREATE_SUSPENDED`. This indicates that the new process's primary thread starts in a suspended state and remains inactive until the ResumeThread function gets invoked.

The `lpCommandLine` parameter of this API call sheds light on the child process that was initiated, namely, `C:\Windows\System32\comp.exe`.

Further down we also notice process injection-related functions being utilized by `discord.exe`.

<img width="1068" height="624" alt="image" src="https://github.com/user-attachments/assets/1cb439cd-d52a-40bf-9bcc-46b47c282899" />

All the above are strong indicators of process injection.

## PowerShell Activity

PowerShell transcripts meticulously log both the commands issued and their respective outputs during a PowerShell session. Occasionally, within a user's documents directory, we might stumble upon PowerShell transcript files. These files grant us a window into the recorded PowerShell activities on the system.

The subsequent screenshot, showcases the PowerShell transcript files nestled within the user's documents directory on a mounted forensic image.

<img width="1481" height="790" alt="image" src="https://github.com/user-attachments/assets/57575385-ea84-48f1-ab67-eef31f02c669" />

Reviewing PowerShell-related activity in detail can be instrumental during investigations.

Here are some recommended guidelines when handling PowerShell data.

- **Unusual Commands:** Look for PowerShell commands that are not typical in your environment or are commonly associated with malicious activities. For example, commands to download files from the internet (Invoke-WebRequest or wget), commands that manipulate the registry, or those that involve creating scheduled tasks.
- **Script Execution:** Check for the execution of PowerShell scripts, especially if they are not signed or come from untrusted sources. Scripts can be used to automate malicious actions.
- **Encoded Commands:** Malicious actors often use encoded or obfuscated PowerShell commands to evade detection. Look for signs of encoded commands in transcripts.
- **Privilege Escalation:** Commands that attempt to escalate privileges, change user permissions, or perform actions typically restricted to administrators can be suspicious.
- **File Operations:** Check for PowerShell commands that involve creating, moving, or deleting files, especially in sensitive system locations.
- **Network Activity:** Look for commands related to network activity, such as making HTTP requests or initiating network connections. These may be indicative of command and control (C2) communications.
- **Registry Manipulation:** Check for commands that involve modifying the Windows Registry, as this can be a common tactic for malware persistence.
- **Use of Uncommon Modules:** If a PowerShell script or command uses uncommon or non-standard modules, it could be a sign of suspicious activity.
- **User Account Activity:** Look for changes to user accounts, including creation, modification, or deletion. Malicious actors may attempt to create or manipulate user accounts for persistence.
- **Scheduled Tasks:** Investigate the creation or modification of scheduled tasks through PowerShell. This can be a common method for persistence.
- **Repeated or Unusual Patterns:** Analyze the patterns of PowerShell commands. Repeated, identical commands or unusual sequences of commands may indicate automation or malicious behavior.
- **Execution of Unsigned Scripts:** The execution of unsigned scripts can be a sign of suspicious activity, especially if script execution policies are set to restrict this.

## 🔑 Key Takeaway

Rapid triage combines filesystem, event log, registry, execution, and API-call artifacts to reconstruct activity on a Windows system. Tools such as Eric Zimmerman's utilities, Timeline Explorer, EvtxECmd, EQL, Registry Explorer, RegRipper, PECmd, and API Monitor help investigators correlate timestamps, identify suspicious execution, uncover persistence mechanisms, and build an evidence-based timeline.
