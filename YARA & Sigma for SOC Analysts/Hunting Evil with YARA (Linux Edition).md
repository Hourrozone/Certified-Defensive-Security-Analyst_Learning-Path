# Hunting Evil with YARA (Linux Edition)

## Introduction

Let's face the reality of cybersecurity operations: often, as Security Analysts, we don’t get the luxury of direct access to a potentially compromised system. Imagine, we've got suspicious flags waving from a remote machine, but due to organizational boundaries, permissions, or logistical issues, we just can't lay our hands on it. It feels like knowing there's a potential fire but not being able to see or touch it directly. This situation can be nerve-wracking, but it's a challenge we've learned to tackle.

Here's where things get interesting. Even if we can't access the machine, in many cases, a memory capture (or memory dump) from the suspicious system can be handed over to us in the Security Operations Center (SOC). It's akin to receiving a snapshot of everything happening in the system at a particular moment. And just because we have this snapshot, doesn't mean our hands are tied.

Luckily, our trusty tool YARA comes to the rescue. We can run YARA-based scans directly on these memory images. It's like having x-ray vision: we can peer into the state of the system, looking for signs of malicious activity or compromised indicators, all without ever having direct access to the machine itself. This capability not only enhances our investigative prowess but also ensures that even remote, inaccessible systems don't remain black boxes to us. So, while the direct path may be blocked, with tools like YARA and our expertise, we always find a way to shine a light into the shadows.

## Hunting for Evil Within Memory Images with YARA

Incorporating YARA extends the capabilities of memory forensics, a pivotal technique in malware analysis and incident response. It equips us to traverse memory content, hunting for telltale signs or compromise indicators.

YARA's memory image scanning mirrors its disk-based counterpart. Let's map out the process:

1. **Create YARA Rules:** Either develop bespoke YARA rules or lean on existing ones that target memory-based malware traits or dubious behaviors.
2. **Compile YARA Rules:** Compile the YARA rules into a binary format using the `yarac` tool (YARA Compiler). This step creates a file containing the compiled YARA rules with a `.yrc` extension. This step is optional, as we can use the normal rules in text format as well. While it is possible to use YARA in its human-readable format, compiling the rules is a best practice when deploying YARA-based detection systems or working with a large number of rules to ensure optimal performance and effectiveness. Also, compiling rules provides some level of protection by converting them into binary format, making it harder for others to view the actual rule content.
3. **Obtain Memory Image:** Capture a memory image using tools such as DumpIt, MemDump, Belkasoft RAM Capturer, Magnet RAM Capture, FTK Imager, and LiME (Linux Memory Extractor).
4. **Memory Image Scanning with YARA:** Use the `yara` tool and the compiled YARA rules to scan the memory image for possible matches.

For instance, we have a memory snapshot named `compromised_system.raw` (residing in the `/home/htb-student/MemoryDumps` directory of this section's target) originating from a system under the siege of WannaCry ransomware. Let's confront this image with the `wannacry_artifacts_memory.yar` YARA rule (residing in the `/home/htb-student/Rules/yara` directory of this section's target).

Here's an example command for YARA-based memory scanning:

<img width="586" height="798" alt="image" src="https://github.com/user-attachments/assets/615bbde3-aa1f-4313-a8e4-656eb3e53378" />

Beyond standalone tools, diving deeper into memory forensics offers a plethora of avenues. Integrating YARA within memory forensics frameworks amplifies its potential. With the Volatility framework and YARA operating in tandem, WannaCry-specific IOCs can be detected seamlessly.

The Volatility framework is a powerful open-source memory forensics tool used to analyze memory images from various operating systems. YARA can be integrated into the Volatility framework as a plugin called `yarascan` allowing for the application of YARA rules to memory analysis.

The Volatility framework is covered in detail inside HTB Academy's Introduction to Digital Forensics module.

For now, let's only discuss how YARA can be used as a plugin in the Volatility framework.

## Single Pattern YARA Scanning Against a Memory Image

In this case, we'll specify a YARA rule pattern directly in the command-line which is searched within the memory image by the `yarascan` plugin of Volatility. The string should be enclosed in quotes (") after the `-U` option. This is useful when we have a specific YARA rule or pattern that we want to apply without creating a separate YARA rules file.

From previous analysis we know that WannaCry malware attempt to connect to the following hard-coded URI `www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com`

Introducing this pattern within the command line using `-U "www.iuqerfsodp9ifjaposdfjhgosurijfaewrwergwea.com"` prompts a search within the `compromised_system.raw` memory image.

<img width="738" height="706" alt="image" src="https://github.com/user-attachments/assets/1620337e-d646-40bd-b8fb-6a4899df5c6a" />

This option allows us to directly specify a YARA rule string within the command-line itself. Let's see how we can search for the content of a whole YARA rule file (i.e. `.yar` rule file) in memory image files.

## Multiple YARA Rule Scanning Against a Memory Image

When we have multiple YARA rules or a set of complex rules that we want to apply to a memory image, we can use the `-y` option followed by the rule file path in the Volatility framework, which allows us to specify the path to a YARA rules file. The YARA rules file (`wannacry_artifacts_memory.yar` in our case) should contain one or more YARA rules in a separate file.

The YARA rules file we will use for demostration purposes is the following.

<img width="729" height="430" alt="image" src="https://github.com/user-attachments/assets/fff456f1-2346-4cdf-ad35-3268d6ee6e4d" />

Let's run Volatility with the rule `wannacry_artifacts_memory.yar` (residing in the `/home/htb-student/Rules/yara` directory) to scan the memory image `compromised_system.raw` (residing in the `/home/htb-student/MemoryDumps` directory)

<img width="744" height="785" alt="image" src="https://github.com/user-attachments/assets/30cd22e5-bdd6-4f3a-844c-b9130610326d" />

We can see in the results that the `yarascan` plugin in Volatility is able to find the process `svchost.exe` with PID `1576` in the memory image of the compromised system.

In summary, the `-U` option allows us to directly specify a YARA rule string within the command-line, while the `-y` option is used to specify the path to a file containing one or more YARA rules. The choice between the two options depends on our specific requirements and whether we have a single rule or a set of rules to apply during the analysis.

## 🔑 Key Takeaway

YARA can scan memory images obtained from potentially compromised systems, allowing analysts to hunt for malware indicators even without direct access to the original machine. It can be used directly with memory images or integrated into Volatility through the `yarascan` plugin. Use `-U` to search for a single pattern supplied on the command line, or `-y` to apply one or more rules from a YARA rule file.
