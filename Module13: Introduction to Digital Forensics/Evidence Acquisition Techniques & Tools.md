<img width="1999" height="704" alt="image" src="https://github.com/user-attachments/assets/471c2141-1f49-46f1-a323-0e078e28a493" /><img width="1129" height="634" alt="image" src="https://github.com/user-attachments/assets/247d73b6-d1d9-479d-9f26-b179688b87a6" /># Evidence Acquisition Techniques & Tools

Evidence acquisition is a critical phase in digital forensics, involving the collection of digital artifacts and data from various sources to preserve potential evidence for analysis. This process requires specialized tools and techniques to ensure the integrity, authenticity, and admissibility of the collected evidence. Here's an overview of evidence acquisition techniques commonly used in digital forensics:

1. Forensic Imaging
2. Extracting Host-based Evidence & Rapid Triage
3. Extracting Network Evidence

## Forensic Imaging

Forensic imaging is a fundamental process in digital forensics that involves creating an exact, bit-by-bit copy of digital storage media, such as hard drives, solid-state drives, USB drives, and memory cards. This process is crucial for preserving the original state of the data, ensuring data integrity, and maintaining the admissibility of evidence in legal proceedings. Forensic imaging plays a critical role in investigations by allowing analysts to examine evidence without altering or compromising the original data.

### Forensic Imaging Tools and Solutions

Below are some forensic imaging tools and solutions:

- **FTK Imager:** Developed by AccessData (now acquired by Exterro), FTK Imager is one of the most widely used disk imaging tools in the cybersecurity field. It allows us to create perfect copies (or images) of computer disks for analysis, preserving the integrity of the evidence. It also lets us view and analyze the contents of data storage devices without altering the data.
- **AFF4 Imager:** A free, open-source tool crafted for creating and duplicating forensic disk images. It's user-friendly and compatible with numerous file systems. A benefit of the AFF4 Imager is its capability to extract files based on their creation time, segment volumes, and reduce the time taken for imaging through compression.
- **DD and DCFLDD:** Both are command-line utilities available on Unix-based systems (including Linux and MacOS). DD is a versatile tool included in most Unix-based systems by default, while DCFLDD is an enhanced version of DD with features specifically useful for forensics, such as hashing.
- **Virtualization Tools:** Given the prevalent use of virtualization in modern systems, incident responders will often need to collect evidence from virtual environments. Depending on the specific virtualization solution, evidence can be gathered by temporarily halting the system and transferring the directory that houses it. Another method is to utilize the snapshot capability present in numerous virtualization software tools.

### Example 1: Forensic Imaging with FTK Imager

Let's now see a demonstration of utilizing FTK Imager to craft a disk image. Be mindful that you'll require an auxiliary storage medium, like an external hard drive or USB flash drive, to save the resultant disk image.

1. Select File -> Create Disk Image.

<img width="1129" height="634" alt="image" src="https://github.com/user-attachments/assets/1e8f769d-f998-48a4-9061-dfac7c61605b" />

2. Next, select the media source. Typically, it's either Physical Drive or Logical Drive.

<img width="434" height="350" alt="image" src="https://github.com/user-attachments/assets/85aa0fba-bd9c-485e-ae7f-09f87af6ab41" />

3. Choose the drive from which you wish to create an image.

<img width="434" height="345" alt="image" src="https://github.com/user-attachments/assets/a901cb8a-86df-4f88-9470-13ea20b6c024" />

4. Specify the destination for the image.

<img width="429" height="393" alt="image" src="https://github.com/user-attachments/assets/aa6b0e98-821e-4ec6-b3c2-3d4e3def2586" />

5. Select the desired image type.

<img width="433" height="326" alt="image" src="https://github.com/user-attachments/assets/e2a02612-658a-4a63-ae53-55ed8fc213c2" />

6. Input evidence details.

<img width="429" height="325" alt="image" src="https://github.com/user-attachments/assets/3891d12c-09bd-476a-8f5f-329ea4781040" />

7. Choose the destination folder and filename for the image. At this step, you can also adjust settings for image fragmentation and compression.

<img width="433" height="322" alt="image" src="https://github.com/user-attachments/assets/56d3e767-608f-471d-b6b4-6276c2188541" />

8. Once all settings are confirmed, click Start.

<img width="424" height="391" alt="image" src="https://github.com/user-attachments/assets/aff0fedd-68e9-4942-b5d9-2ffcf2d6d8fb" />

9. You'll observe the progress of the imaging.

<img width="397" height="288" alt="image" src="https://github.com/user-attachments/assets/7d0f0aab-8d1a-4e86-8696-99ed04f2d46d" />

10. If you opted to verify the image, you'll also see the verification progress.

<img width="396" height="246" alt="image" src="https://github.com/user-attachments/assets/a58fb1ec-9d1b-47d7-91b7-bf5a43d3387d" />

11. After the image has been verified, you'll receive an imaging summary. Now, you're prepared to analyze this dump.

<img width="488" height="374" alt="image" src="https://github.com/user-attachments/assets/747b52dd-3917-4303-a920-994b062925c3" />

### Example 2: Mounting a Disk Image with Arsenal Image Mounter

Let's now see another demonstration of utilizing Arsenal Image Mounter to mount a disk image we have previously created (not the one mentioned above) from a compromised Virtual Machine (VM) running on VMWare. The virtual hard disk of the VM has been stored as `HTBVM01-000003.vmdk`.

After we've installed Arsenal Image Mounter, let's ensure we launch it with administrative rights. In the main window of Arsenal Image Mounter, let's click on the Mount disk image button. From there, we'll navigate to the location of our .VMDK file and select it.

<img width="996" height="553" alt="image" src="https://github.com/user-attachments/assets/8a379bb2-1358-4835-a32f-93f2762a54b4" />


Arsenal Image Mounter will then start its analysis of the VMDK file. We'll also have the choice to decide if we want to mount the disk as read-only or read-write, based on our specific requirements.

Choosing to mount a disk image as read-only is a foundational step in digital forensics and incident response. This approach is vital for preserving the original state of evidence, ensuring its authenticity and integrity remain intact.

Once mounted, the image will appear as a drive, assigned the letter `D:\`.

<img width="1422" height="588" alt="image" src="https://github.com/user-attachments/assets/adf3cfc4-6b6a-4cbf-9cb5-6f20c0dc3acb" />

## Extracting Host-based Evidence & Rapid Triage

### Host-based Evidence

Modern operating systems, with Microsoft Windows being a prime example, generate a plethora of evidence artifacts. These can arise from application execution, file modifications, or even the creation of user accounts. Each of these actions leaves behind a trail, providing invaluable insights for incident response analysts.

Evidence on a host system varies in its nature. The term volatility refers to the persistence of data on a host system, with volatile data being information that disappears after events such as logoffs or power shutdowns. One crucial type of volatile evidence is the system's active memory. During investigations, especially those concerning malware infections, this live system memory becomes indispensable. Malware often leaves traces within system memory, and losing this evidence can hinder an analyst's investigation. To capture memory, tools like FTK Imager are commonly employed.

#### Memory Acquisition Solutions

Some other memory acquisition solutions are:

- **WinPmem:** WinPmem has been the default open source memory acquisition driver for windows for a long time. It used to live in the Rekall project, but has recently been separated into its own repository.
- **DumpIt:** A simplistic utility that generates a physical memory dump of Windows and Linux machines. On Windows, it concatenates 32-bit and 64-bit system physical memory into a single output file, making it extremely easy to use.
- **MemDump:** MemDump is a free, straightforward command-line utility that enables us to capture the contents of a system's RAM. It’s quite beneficial in forensics investigations or when analyzing a system for malicious activity. Its simplicity and ease of use make it a popular choice for memory acquisition.
- **Belkasoft RAM Capturer:** This is another powerful tool we can use for memory acquisition, provided free of charge by Belkasoft. It can capture the RAM of a running Windows computer, even if there's active anti-debugging or anti-dumping protection. This makes it a highly effective tool for extracting as much data as possible during a live forensics investigation.
- **Magnet RAM Capture:** Developed by Magnet Forensics, this tool provides a free and simple way to capture the volatile memory of a system.
- **LiME (Linux Memory Extractor):** LiME is a Loadable Kernel Module (LKM) which allows the acquisition of volatile memory. LiME is unique in that it's designed to be transparent to the target system, evading many common anti-forensic measures.

### Example 1: Acquiring Memory with WinPmem

Let's now see a demonstration of utilizing WinPmem for memory acquisition.

To generate a memory dump, simply execute the command below with Administrator privileges.

<img width="953" height="591" alt="image" src="https://github.com/user-attachments/assets/2df25604-440f-4ffd-be2e-274b919a25e9" />

### Example 2: Acquiring VM Memory

Here are the steps to acquire memory from a Virtual Machine (VM).

1. Open the running VM's options
2. Suspend the running VM
3. Locate the .vmem file inside the VM's directory.

<img width="1126" height="494" alt="image" src="https://github.com/user-attachments/assets/74e467e0-7e5b-4ab1-9719-907d724f48eb" />

<img width="916" height="454" alt="image" src="https://github.com/user-attachments/assets/2e6a203c-89b8-4412-bc68-4b9ad2b1092a" />

On the other hand, non-volatile data remains on the hard drive, typically persisting through shutdowns. This category includes artifacts such as:

- Registry
- Windows Event Log
- System-related artifacts (e.g., Prefetch, Amcache)
- Application-specific artifacts (e.g., IIS logs, Browser history)

### Rapid Triage

This approach emphasizes collecting data from potentially compromised systems. The goal is to centralize high-value data, streamlining its indexing and analysis. By centralizing this data, analysts can more effectively deploy tools and techniques, honing in on systems with the most evidentiary value. This targeted approach allows for a deeper dive into digital forensics, offering a clearer picture of the adversary's actions.

One of the best, if not the best, rapid artifact parsing and extraction solutions is KAPE (Kroll Artifact Parser and Extractor). Let's see how we can employ KAPE to retrieve valuable forensic data from the image we previously mounted with the help of Arsenal Image Mounter (`D:\`).

KAPE is a powerful tool in the realm of digital forensics and incident response. Designed to aid forensic experts and investigators, KAPE facilitates the collection and analysis of digital evidence from Windows-based systems. Developed and maintained by Kroll, KAPE is celebrated for its comprehensive collection features, adaptability, and intuitive interface. The diagram below illustrates KAPE's operational flow.

<img width="1579" height="419" alt="image" src="https://github.com/user-attachments/assets/468a8575-31ef-4466-b878-fd2ea15efff1" />

**Image reference:** https://ericzimmerman.github.io/KapeDocs/#!index.md

KAPE operates based on the principles of Targets and Modules. These elements guide the tool in processing data and extracting forensic artifacts. When we feed a source to KAPE, it duplicates specific forensic-related files to a designated output directory, all while maintaining the metadata of each file.

<img width="1917" height="1076" alt="image" src="https://github.com/user-attachments/assets/62dead4b-be35-4560-a4a1-aa67a121e54c" />

After downloading, let's unzip the file and launch KAPE. Within the KAPE directory, we'll notice two executable files: `gkape.exe` and `kape.exe`. KAPE provides users with two modes: CLI (`kape.exe`) and GUI (`gkape.exe`).

<img width="804" height="310" alt="image" src="https://github.com/user-attachments/assets/0342df0b-7a77-48c0-a21c-142273702606" />

Let's opt for the GUI version to explore the available options more visually.

<img width="1265" height="912" alt="image" src="https://github.com/user-attachments/assets/e96a946a-43c4-4da8-9cf3-f2d0a950027a" />

The crux of the process lies in selecting the appropriate target configurations.

<img width="984" height="258" alt="image" src="https://github.com/user-attachments/assets/a97fbede-b2ca-49e8-848d-4097b9c53740" />

In KAPE's terminology, Targets refer to the specific artifacts we aim to extract from an image or system. These are then duplicated to the output directory.

KAPE's target files have a `.tkape` extension and reside in the `<path to kape>\KAPE\Targets` directory. For instance, the target `RegistryHivesSystem.tkape` in the screenshot below specifies the locations and file masks associated with system-related registry hives. In this target configuration, `RegistryHivesSystem.tkape` contains information to collect the files with file mask `SAM.LOG*` from the `C:\Windows\System32\config` directory.

<img width="1243" height="508" alt="image" src="https://github.com/user-attachments/assets/1975ea14-1711-47ea-93fa-412e2ae6af96" />

KAPE also offers Compound Targets, which are essentially amalgamations of multiple targets. This feature accelerates the collection process by gathering multiple files defined across various targets in a single run. The Compound directory's KapeTriage file provides an overview of the contents of this compound target.

<img width="1443" height="650" alt="image" src="https://github.com/user-attachments/assets/f4903db8-ee2f-432b-9bec-e572b54fa842" />

Let's specify our source (in our scenario, it's `D:\`) and designate a location to store the harvested data. We can also determine an output folder to house the processed data from KAPE.

After configuring our options, let's hit the Execute button to initiate the data collection.

<img width="1265" height="915" alt="image" src="https://github.com/user-attachments/assets/ffddc142-8865-47d3-9878-296c6017898f" />

Upon execution, KAPE will commence the collection, storing the results in the predetermined destination.

<img width="813" height="768" alt="image" src="https://github.com/user-attachments/assets/ef90abbd-cc10-435c-b170-5c16ab6f016f" />

The output directory of KAPE houses the fruits of the artifact collection and processing. The exact contents of this directory can differ based on the artifacts selected and the configurations set. In our demonstration, we opted for the `!SANS_Triage` collection target configuration. Let's navigate to KAPE's output directory to inspect the harvested data.

<img width="933" height="337" alt="image" src="https://github.com/user-attachments/assets/9d76fa20-db56-4b81-abf0-a6cf668d83d8" />

From the displayed results, it's evident that the `$MFT` file has been collected, along with the Users and Windows directories.

It's worth noting that KAPE has also harvested the Windows event logs, which are nestled within the Windows directory sub-folders.

<img width="974" height="424" alt="image" src="https://github.com/user-attachments/assets/2455f017-a5d4-44cd-aa2f-7cb3bb7eac33" />

What if we wanted to perform artifact collection remotely and en masse? This is where EDR solutions and Velociraptor come into play.

#### EDR Solutions

Endpoint Detection and Response (EDR) platforms offer a significant advantage for incident response analysts. They enable remote acquisition and analysis of digital evidence. For instance, EDR platforms can display recently executed binaries or newly added files. Instead of sifting through individual systems, analysts can search for such indicators across the entire network. Another benefit is the capability to gather evidence, be it specific files or comprehensive forensic packages. This functionality expedites evidence collection and facilitates large-scale searching and collection.

#### Velociraptor

Velociraptor is a potent tool for gathering host-based information using Velociraptor Query Language (VQL) queries. Beyond this, Velociraptor can execute Hunts to amass various artifacts. A frequently utilized artifact is the `Windows.KapeFiles.Targets`. While KAPE (Kroll Artifact Parser and Extractor) itself isn't open-source, its file collection logic, encoded in YAML, is accessible via the KapeFiles project. This approach is a staple in Rapid Triage.

**To utilize Velociraptor for KapeFiles artifacts:**

1. Initiate a new Hunt

<img width="3813" height="647" alt="image" src="https://github.com/user-attachments/assets/42291429-b56e-4b3d-8b32-91e00f258a19" />

<img width="1983" height="719" alt="image" src="https://github.com/user-attachments/assets/c9875093-5a2b-426d-af68-17f5f37e67f0" />


2. Choose `Windows.KapeFiles.Targets` as the artifacts for collection.

<img width="1981" height="719" alt="image" src="https://github.com/user-attachments/assets/c18743ba-d51d-477e-82f6-8d7ec5c2746c" />

3. Specify the collection to use.

<img width="1914" height="765" alt="image" src="https://github.com/user-attachments/assets/4d1576cb-3f82-4324-b8dc-b86a6261c4d0" />

<img width="1977" height="720" alt="image" src="https://github.com/user-attachments/assets/6fa08b64-33c1-4779-8b50-a97e96044e7b" />


4. Click on Launch to start the hunt.

<img width="1999" height="704" alt="image" src="https://github.com/user-attachments/assets/06959a80-11b8-4edb-9f5e-6d26ed376d1f" />

5. Once completed, download the results.

<img width="1999" height="706" alt="image" src="https://github.com/user-attachments/assets/70e61760-8865-4ad4-9f07-d835fdc5fe51" />

6. Extracting the archive will reveal files related to the collected artifacts and all gathered files.

<img width="870" height="584" alt="image" src="https://github.com/user-attachments/assets/0d5a0d82-ca00-45c1-a688-308112ab5a8b" />

<img width="885" height="586" alt="image" src="https://github.com/user-attachments/assets/fe6bfb80-f8dd-4f47-8df3-9c1d74d9e7c9" />

**For remote memory dump collection using Velociraptor:**

1. Start a new Hunt, but this time, select the `Windows.Memory.Acquisition` artifact.

<img width="1970" height="703" alt="image" src="https://github.com/user-attachments/assets/1ea05d13-b91f-48c3-8b71-a3cb01f6a86a" />

2. After the Hunt concludes, download the resulting archive. Within, you'll find a file named `PhysicalMemory.raw`, containing the memory dump.

<img width="1999" height="735" alt="image" src="https://github.com/user-attachments/assets/c23ecae7-703a-417e-86d9-2a46a63cf886" />

## Extracting Network Evidence

Throughout our exploration of the modules in the SOC Analyst path, we've delved extensively into the realm of network evidence, a fundamental aspect for any SOC analyst.

First up, our Intro to Network Traffic Analysis and Intermediate Network Traffic Analysis modules covered traffic capture analysis. Think of traffic capture as a snapshot of all the digital conversations happening in our network. Tools like Wireshark or tcpdump allow us to capture and dissect these packets, giving us a granular view of data in transit.

Then, our Working with IDS/IPS and Detecting Windows Attacks with Splunk modules covered the usage of IDS/IPS-derived data. Intrusion Detection Systems (IDS) are our watchful sentinels, constantly monitoring network traffic for signs of malicious activity. When they spot something amiss, they alert us. On the other hand, Intrusion Prevention Systems (IPS) take it a step further. Not only do they detect, but they also take pre-defined actions to block or prevent those malicious activities.

Traffic flow data, often sourced from tools like NetFlow or sFlow, provides us with a broader view of our network's behavior. While it might not give us the nitty-gritty details of each packet, it offers a high-level overview of traffic patterns.

Lastly, our trusty firewalls. These are not just barriers that block or allow traffic based on predefined rules. Modern firewalls are intelligent beasts. They can identify applications, users, and even detect and block threats. By analyzing firewall logs, we can uncover attempts to exploit vulnerabilities, unauthorized access attempts, and other malicious activities.

## 🔑 Key Takeaway

Evidence acquisition preserves digital evidence for forensic investigation through forensic imaging, memory capture, host-based artifact collection, rapid triage, and network evidence gathering. Tools such as FTK Imager, Arsenal Image Mounter, WinPmem, KAPE, and Velociraptor help collect evidence while maintaining its integrity. Network captures, IDS/IPS data, traffic flow records, and firewall logs provide additional visibility into suspicious activity.
