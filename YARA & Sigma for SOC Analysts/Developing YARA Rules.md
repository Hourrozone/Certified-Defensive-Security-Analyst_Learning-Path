<img width="748" height="794" alt="image" src="https://github.com/user-attachments/assets/aec809fe-d922-4dd1-9b42-673342eae83a" /># Developing YARA Rules

In this section, we'll cover manual and automated YARA rule development.

Let's dive into the world of YARA rules using a sample named `svchost.exe` residing in the `/home/htb-student/Samples/YARASigma` directory of this section's target as an illustration. We want to understand the process behind crafting a YARA rule, so let's get our hands dirty.

Initially, we need to conduct a string analysis on our malware sample.

<img width="747" height="599" alt="image" src="https://github.com/user-attachments/assets/93a6e105-0f97-4d15-9c6b-cbd8d133f906" />

From the first few strings, it becomes evident that the file is packed using the UPX (Ultimate Packer for eXecutables) packer. Given this discovery, we can incorporate UPX-related strings to formulate a basic YARA rule targeting samples packed via UPX.

<img width="740" height="326" alt="image" src="https://github.com/user-attachments/assets/fde9ea27-8238-434d-83be-136fc68e3e3d" />

Here's a brief breakdown of our YARA rule crafted for detecting UPX-packed executables:

- **Rule Name:** `UPX_packed_executable`
- **Meta Description:** Provides a description of the rule, stating that it detects UPX-packed executables.
- **Strings Section:** `strings` defines the strings that the rule will search for within the files.
  - `$string_1 = "UPX0"`: Matches the string UPX0 within the file.
  - `$string_2 = "UPX1"`: Matches the string UPX1 within the file.
  - `$string_3 = "UPX2"`: Matches the string UPX2 within the file.
- **Condition:** `condition` specifies the criteria that must be met for the rule to trigger a match.
  - **all of them:** Specifies that all the defined strings (`$string_1`, `$string_2`, and `$string_3`) must be found within the file.

In essence, our `UPX_packed_executable` rule (located inside this section's target at `/home/htb-student/Rules/yara/upx_packed.yar`) scans for the strings UPX0, UPX1, and UPX2 inside a file. If the rule finds all three strings, it raises an alert, hinting that the file might be packed with the UPX packer. This rule is a handy tool when we're on the lookout for executables that have undergone compression or obfuscation using the UPX method.

## Developing a YARA Rule Through yarGen

Let's continue our dive into the world of YARA rules using a sample named `dharma_sample.exe` residing in the `/home/htb-student/Samples/YARASigma` directory of this section's target.

Once again, we need to conduct a string analysis on our malware sample.

<img width="741" height="645" alt="image" src="https://github.com/user-attachments/assets/72f453c0-8a72-4a57-8448-dd20c13f1101" />

After we execute the `strings` command on `dharma_sample.exe`, we spot `C:\crysis\Release\PDB\payload.pdb`, which is pretty unique. Alongside other distinct strings, we can craft a more refined YARA rule. Let's employ yarGen to expedite this process.

yarGen is our go-to tool when we need an automatic YARA rule generator. What makes it a gem is its ability to churn out YARA rules based on strings found in malicious files while sidestepping strings common in benign software. This is possible because yarGen comes equipped with a vast database of goodware strings and opcodes. Before diving in, we need to unpack the ZIP archives containing these databases.

Here's how we get yarGen up and running:

1. Download the latest release from the release section
2. Install all dependencies with `pip install -r requirements.txt`
3. Run `python yarGen.py --update` to automatically download the built-in databases. They will be saved into the `'./dbs'` subfolder (Download: 913 MB).
4. See help with `python yarGen.py --help` for more information on the command line parameters.

**Note:** yarGen can be found inside the `/home/htb-student/yarGen-0.23.4` directory of this section's target.

Let's place our sample in a temp directory (there is one available at `/home/htb-student/temp` inside this section's target) and specify the path using the following command-line arguments.

<img width="748" height="794" alt="image" src="https://github.com/user-attachments/assets/c121d7b8-68ea-422f-bab3-e3edee29213d" />

### Command Breakdown

- **`yarGen.py`:** This is the name of the yarGen Python script that will be executed.
- **`-m /home/htb-student/temp`:** This option specifies the source directory where the sample files (e.g., malware or suspicious files) are located. The script will analyze these samples to generate YARA rules.
- **`-o htb_sample.yar`:** This option indicates the output file name for the generated YARA rules. In this case, the YARA rules will be saved to a file named `htb_sample.yar`.

The resulting YARA rules will be written to the `htb_sample.yar` file inside the `/home/htb-student/yarGen-0.23.4` directory of this section's target. Let's see the content of the generated rule.

<img width="753" height="774" alt="image" src="https://github.com/user-attachments/assets/6c209a83-a3bf-4454-a951-16f5146d33d9" />

Now, for the moment of truth. We'll unleash YARA with our newly minted rule to see if it sniffs out any matches when run against a malware sample repository located at `home/htb-student/Samples/YARASigma` inside this section's target.

<img width="738" height="153" alt="image" src="https://github.com/user-attachments/assets/eace15b5-7c1b-4cca-a1a1-fc88a1e7bda2" />

As we can see, the `pdf_reader.exe`, `microsoft.com`, `check_updates.exe`, and `KB5027505.exe` files are detected by this rule (in addition to `dharma_sample.exe` of course).

## Manually Developing a YARA Rule

### Example 1: ZoxPNG RAT Used by APT17

Let's now go a bit deeper...

We want to develop a YARA rule to scan for a specific variation of the ZoxPNG RAT used by APT17 based on:

- A sample named `legit.exe` residing in the `/home/htb-student/Samples/YARASigma` directory of this section's target
- A post from Intezer
- String analysis
- Imphash
- Common sample file size

Let's start with our string analysis endeavors as follows.

<img width="747" height="787" alt="image" src="https://github.com/user-attachments/assets/74d113cc-11ff-4454-8e23-11e80170b9f9" />

Let's then use the hashes mentioned in Intezer's post to identify common sample sizes. It looks like there are no related samples whose size is bigger than 200KB. An example of an identified sample is the following. https://www.hybrid-analysis.com/sample/ee362a8161bd442073775363bf5fa1305abac2ce39b903d63df0d7121ba60550.

Finally, the sample's Imphash can be calculated as follows, using the `imphash_calc.py` script that resides in the `/home/htb-student` directory of this section's target.

<img width="747" height="89" alt="image" src="https://github.com/user-attachments/assets/59c0e2d4-7d4e-48de-978b-602ee31eb58f" />

A good YARA rule to detect the aforementioned variation of ZoxPNG resides in the `/home/htb-student/Rules/yara` directory of this section's target, saved as `apt_apt17_mal_sep17_2.yar`.

<img width="624" height="839" alt="image" src="https://github.com/user-attachments/assets/afcc7d21-66e9-4141-b3bb-76f933148cf7" />

#### YARA Rule Breakdown

**Rule Imports:** Modules are extensions to YARA's core functionality.

- `import "pe"`: By importing the PE module the YARA rule gains access to a set of specialized functions and structures that can inspect and analyze the details of PE files. This makes the rule more precise when it comes to detecting characteristics in Windows executables.

**Rule Meta:**

- **description:** Tells us the main purpose of the rule, which is to detect APT17 malware.
- **license:** Points to the location and version of the license governing the use of this YARA rule.
- **author:** The rule was written by Florian Roth from Nextron Systems.
- **reference:** Provides a link that goes into more detail about the malware or context of this rule.
- **date:** The date the rule was either created or last updated, in this case, 3rd October 2017.
- **hash1, hash2, hash3:** Hash values, probably of samples related to APT17, which the author used as references or as foundational data to create the rule.

**Rule Body:** The rule contains a series of strings, which are potential indicators of the APT17 malware. These strings are split into two categories

- `$x*` strings
- `$s*` strings

**Rule Condition:** This is the heart of the rule, where the actual detection logic resides.

- `uint16(0) == 0x5a4d`: Checks if the first two bytes of the file are MZ, which is the magic number for Windows executables. So, we're focusing on detecting Windows binaries.
- `filesize < 200KB`: Limits the rule to scan only small files, specifically those smaller than 200KB.
- `pe.imphash() == "414bbd566b700ea021cfae3ad8f4d9b9"`: This checks the import hash (imphash) of the PE (Portable Executable) file. Imphashes are great for categorizing and clustering malware samples based on the libraries they import.
- `1 of ($x*)`: At least one of the `$x` strings (from the strings section) must be present in the file.
- `6 of them`: Requires that at least six of the strings (from both `$x` and `$s` categories) be found within the scanned file.

### Example 2: Neuron Used by Turla

We want to develop a YARA rule to scan for instances of Neuron Service used by Turla based on:

- A sample named `Microsoft.Exchange.Service.exe` residing in the `/home/htb-student/Samples/YARASigma` directory of this section's target
- An analysis report from the National Cyber Security Centre

Since the report mentions that both the Neuron client and Neuron service are written using the .NET framework we will perform .NET "reversing" instead of string analysis.

This can be done using the `monodis` tool as follows.

<img width="660" height="829" alt="image" src="https://github.com/user-attachments/assets/fc0fc60f-0616-426e-98ef-3b77739c1a2d" />

By going through the above we can identify functions and classes within the .NET assembly.

**Note:** A better reversing solution would be to load the .NET assembly (`Microsoft.Exchange.Service.exe`) into a .NET debugger and assembly editor like dnSpy.

<img width="2970" height="1819" alt="image" src="https://github.com/user-attachments/assets/2ea6e128-e6cf-4194-bbb8-44db5fe14d54" />

A good YARA rule to identify instances of Neuron Service resides in the `/home/htb-student/Rules/yara` directory of this section's target, saved as `neuron_1.yar`.

<img width="827" height="680" alt="image" src="https://github.com/user-attachments/assets/d5288cec-fcf2-4745-b813-842446435ce2" />

#### YARA Rule Breakdown

**Strings Section:**

- `$class1 = "StorageUtils" ascii` to `$class8 = "W3WPDIAG" ascii`: These are eight ASCII strings corresponding to class names within the .NET assembly.
- `$func1 = "AddConfigAsString" ascii` to `$func7 = "FindSPath" ascii`: These seven ASCII strings represent class or function names within the .NET assembly.
- `$dotnetMagic = "BSJB" ascii`: This signature is present in the CLI (Common Language Infrastructure) header of .NET binaries, and its presence can be used to indicate the file is a .NET assembly. Specifically, it's in the Signature field of the CLI header, which follows the PE header and additional tables.

**Condition Section:**

- `uint16(0) == 0x5A4D`: This checks if the first two bytes at the start of the file are MZ, a magic number indicating a Windows Portable Executable (PE) format.
- `uint16(uint32(0x3c)) == 0x4550`: A two-step check. First, it reads a 32-bit (4 bytes) value from offset `0x3c` of the file. In PE files, this offset typically contains a pointer to the PE header. It then checks whether the two bytes at that pointer are PE (0x4550), indicating a valid PE header. This ensures the file is a legitimate PE format and not a corrupted or obfuscated one.
- `$dotnetMagic`: Verifies the presence of the BSJB string. This signature is present in the CLI (Common Language Infrastructure) header of .NET binaries, and its presence can be used to indicate the file is a .NET assembly.
- `6 of them`: This condition states that at least six of the previously defined strings (either classes or functions) must be found within the file. This ensures that even if a few signatures are absent or have been modified, the rule will still trigger if a substantial number remain.

### Example 3: Stonedrill Used in Shamoon 2.0 Attacks

We want to develop a YARA rule to scan for instances of Stonedrill used in Shamoon 2.0 attacks based on:

- A sample named `sham2.exe` residing in the `/home/htb-student/Samples/YARASigma` directory of this section's target
- An analysis report from Kaspersky

The report mentions: ... many samples had one additional encrypted resource with a specific, although non-unique name 101.

Encrypted/compressed/obfuscated in PE files usually means high entropy. We can use the `entropy_pe_section.py` script that resides in the `/home/htb-student` directory of this section's target to check if our sample's resource section contains anything encrypted/compressed as follows.

<img width="823" height="530" alt="image" src="https://github.com/user-attachments/assets/9384de07-1e9f-4121-8599-d609f3e8d5c8" />

We notice that the resource section (`.rsrc`) has high entropy (8.0 is the maximum entropy value). We can take for granted that the resource section contains something suspicious.

A good YARA rule to identify instances of Stonedrill resides in the `/home/htb-student/Rules/yara` directory of this section's target, saved as `stonedrill.yar`.

<img width="748" height="776" alt="image" src="https://github.com/user-attachments/assets/634c9219-0db0-42fc-a9af-858936a094f6" />

#### YARA Rule Breakdown

**Rule Imports:** Modules are extensions to YARA's core functionality.

- `import "pe"`: By importing the PE module the YARA rule gains access to a set of specialized functions and structures that can inspect and analyze the details of PE files. This makes the rule more precise when it comes to detecting characteristics in Windows executables.
- `import "math"`: Imports the math module, providing mathematical functions like entropy calculations.

**Rule Meta:**

- `copyright = "Kaspersky Lab"`: The rule was authored or copyrighted by Kaspersky Lab.
- `description = "Generic detection for samples that enumerate files with encrypted resource called 101"`: The rule aims to detect samples that list files and have an encrypted resource with the identifier "101".
- `reference = "https://securelist.com/from-shamoon-to-stonedrill/77725/"`: Provides an URL for additional context or information about the rule.
- `hash`: Two hashes are given, probably as examples of known malicious files that match this rule.
- `version = "1.4"`: The version number of the YARA rule.

**Strings Section:**

- `$mz = "This program cannot be run in DOS mode."`: The ASCII string that typically appears in the DOS stub part of a PE file.
- `$a1 = "FindFirstFile"`, `$a2 = "FindNextFile"`: Strings for Windows API functions used to enumerate files. The usage of FindFirstFileW and FindNextFileW API functions can be idenfitied through string analysis.
- `$a3 = "FindResource"`, `$a4 = "LoadResource"`: As already mentioned Stonedrill samples feature encrypted resources. These strings can be found through string analysis and they are related to Windows API functions used for handling resources within the executable.

**Rule Condition:**

- `uint16(0) == 0x5A4D`: Checks if the first two bytes of the file are "MZ," indicating a Windows PE file.
- `all of them`: All the strings `$a1`, `$a2`, `$a3`, `$a4` must be present in the file.
- `filesize < 700000`: The file size must be less than 700,000 bytes.
- `pe.number_of_sections > 4`: The PE file must have more than four sections.
- `pe.number_of_signatures == 0`: The file must not be digitally signed.
- `pe.number_of_resources > 1 and pe.number_of_resources < 15`: The file must contain more than one but fewer than 15 resources.
- `for any i in (0..pe.number_of_resources - 1): ( (math.entropy(pe.resources[i].offset, pe.resources[i].length) > 7.8) and pe.resources[i].id == 101 and pe.resources[i].length > 20000 and pe.resources[i].language == 0 and not ($mz in (pe.resources[i].offset..pe.resources[i].offset + pe.resources[i].length)) )`: Go through each resource in the file and check if the entropy of the resource data is more than 7.8 and the resource identifier is 101 and the resource length is greater than 20,000 bytes and the language identifier of the resource is 0 and the DOS stub string is not present in the resource. It's not required for all resources to match the condition; only one resource meeting all the criteria is sufficient for the overall YARA rule to be a match.

## YARA Rule Development Resources

As you can imagine, the best YARA rule development resource is the official documentation, which can be found at the following link.

The next best resource on effective YARA rule developement comes from Kaspersky.

Below are some blog posts that offer a more detailed explanation on how to use yarGen for YARA rule development:

- How to Write Simple but Sound Yara Rules - Part 1
- How to Write Simple but Sound Yara Rules - Part 2
- How to Write Simple but Sound Yara Rules - Part 3

yarGen is a great tool to develop some good yara rules by extracting unique patterns. Once the rule is developed with the help of yarGen, we definitely need to review and add/remove some more patterns to make it an effective rule.

In this blogpost, Florian Roth has mentioned that the main purpose of yarGen is to develop the best possible rules for manual post-processing which might sound like a tedious task, but the combination of clever automatic preselection and a critical human analyst beats both the fully manual and fully automatic generation process.

Continuing forward, we'll dig deeper into using YARA, expanding our hunt for threats from our filesystem to memory and also within memory images.

## 🔑 Key Takeaway

Developing effective YARA rules involves identifying distinctive strings, file characteristics, hashes, and structural indicators from malware samples. Tools like yarGen can automate initial rule generation, while manual analysis and refinement help improve accuracy and reduce false positives. YARA rules can combine string matches, PE metadata, file size, entropy, and other conditions to detect specific malware families and variants.
