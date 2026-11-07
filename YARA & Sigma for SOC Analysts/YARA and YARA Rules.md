# YARA and YARA Rules

YARA is a powerful pattern-matching tool and rule format used for identifying and classifying files based on specific patterns, characteristics, or content. SOC analysts commonly use YARA rules to detect and classify malware samples, suspicious files, or indicators of compromise (IOCs).

YARA rules are typically written in a rule syntax that defines the conditions and patterns to be matched within files. These rules can include various elements, such as strings, regular expressions, and Boolean logic operators, allowing analysts to create complex and precise detection rules. It's important to note that YARA rules can recognize both textual and binary patterns, and they can be applied to memory forensics activities as well.

When applied, YARA scans files or directories and matches them against the defined rules. If a file matches a specific pattern or condition, it can trigger an alert or warrant further examination as a potential security threat.

<img width="1476" height="311" alt="image" src="https://github.com/user-attachments/assets/bc03ce85-895d-456d-a010-3f4e03747b5b" />

YARA rules are especially useful for SOC analysts when analyzing malware samples, conducting forensic investigations, or performing threat hunting activities. The flexibility and extensibility of YARA make it a valuable tool in the cybersecurity community.

## Usages of Yara

### Malware Detection and Classification

YARA is commonly used for detecting and identifying malware samples based on specific patterns, characteristics, or indicators. YARA rules can be created to match against known malware signatures, behaviors, or file properties, aiding in the identification of malicious files and potentially preventing further compromise. Within the scope of digital forensics, YARA can also identify suspicious or malicious patterns in captured memory images.

### File Analysis and Classification

YARA is valuable for analyzing and classifying files based on specific patterns or attributes. Analysts can create YARA rules to categorize files by different file formats or file type, version, metadata, packers or other characteristics. This capability is useful in forensic analysis, malware research, or identifying specific file types within large datasets.

### Indicator of Compromise (IOC) Detection

YARA can be employed to search for specific indicators of compromise (IOCs) within files or directories. By defining YARA rules that target specific IOC patterns, such as file names, registry keys, or network artifacts, security teams can identify potential security breaches or ongoing attacks.

### Community-driven Rule Sharing

With YARA, we have the ability to tap into a community that regularly contributes and shares their detection rules. This ensures that we constantly update and refine our detection mechanisms.

### Create Custom Security Solutions

By combining YARA rules with other techniques like static and dynamic analysis, sandboxing, and behavior monitoring, effective security solutions can be created.

### Custom Yara Signatures/Rules

YARA allows us to create custom rules tailored to our organization's specific needs and environment. By deploying these custom rules in our security infrastructure, such as antivirus or endpoint detection and response (EDR) solutions, we can enhance our defense capabilities. Custom YARA rules can help identify unique or targeted threats that are specific to our organization's assets, applications, or industry.

### Incident Response

YARA aids in incident response by enabling analysts to quickly search and analyze files or memory images for specific patterns or indicators. By applying YARA rules during the investigation process, analysts can identify relevant artifacts, determine the scope of an incident, and gather critical information to aid in remediation efforts.

### Proactive Threat Hunting

Instead of waiting for an alert, we can leverage YARA to perform proactive searches across our environments, searching for potential threats or remnants of past infections.

## How Does YARA Work?

In summary, the YARA scan engine, equipped with YARA modules, scans a set of files by comparing their content against the patterns defined in a set of rules. When a file matches the patterns and conditions specified in a YARA rule, it is considered a detected file. This process allows analysts to efficiently identify files that exhibit specific behaviors or characteristics, aiding in malware detection, IOC identification, and threat hunting. This flow is demonstrated in the diagram below.

<img width="1718" height="663" alt="image" src="https://github.com/user-attachments/assets/81a1479b-9733-46e8-8754-08aaf5334f34" />

In the above diagram, we can see that the YARA scan engine, using YARA modules, matches patterns defined in a set of rules against a set of files, resulting in the detection of files that meet the specified patterns and conditions. This is how YARA rules are helpful to identify threats. Let's understand this in more detail.

### Set of Rules (containing suspicious patterns)

First of all, we have one or more YARA rules, which are created by security analysts. These rules define specific patterns, characteristics, or indicators that need to be matched within files. Rules can include strings, regular expressions, byte sequences, and other indicators of interest. The rules are typically stored in a YARA rule file format (e.g., `.yara` or `.yar` file) for easy management and reuse.

### Set of Files (for scanning)

A set of files, such as executables, documents, or other binary or text-based files, are provided as input to the YARA scan engine. The files can be stored on a local disk, within a directory, or even within memory images or network traffic captures.

### YARA Scan Engine

The YARA scan engine is the core component responsible for performing the actual scanning and matching of files against the defined YARA rules. It utilizes YARA modules, which are sets of algorithms and techniques, to efficiently compare the content of files against the patterns specified in the rules.

### Scanning and Matching

The YARA scan engine iterates through each file in the set, one at a time. For each file, it analyzes the content byte by byte, looking for matches against the patterns defined in the YARA rules. The YARA scan engine uses various matching techniques, including string matching, regular expressions, and binary matching, to identify patterns and indicators within the files.

### Detection of Files

When a file matches the patterns and conditions specified in a YARA rule, it is considered a detected file. The YARA scan engine records information about the match, such as the matched rule, the file path, and the offset within the file where the match occurred and provides output indicating the detection, which can be further processed, logged, or used for subsequent actions.

## YARA Rule Structure

Let's dive into the structure of a YARA rule. YARA rules consist of several components that define the conditions and patterns to be matched within files. An example of a YARA rule is as follows:

<img width="746" height="358" alt="image" src="https://github.com/user-attachments/assets/a2c07afe-7a5a-4df1-8568-83edd9d188a0" />

Each rule in YARA starts with the keyword `rule` followed by a rule identifier. Rule identifiers are case sensitive where the first character cannot be a digit, and cannot exceed 128 characters.

The following keywords are reserved and cannot be used as an identifier:

<img width="1366" height="456" alt="image" src="https://github.com/user-attachments/assets/000f1feb-397e-41e5-88ec-8ec79ccc5134" />

Now, let's go over the YARA rule structure using a rule that identifies strings associated with the WannaCry ransomware as an example. The rule below instructs YARA to flag any file containing all three specified strings as `Ransomware_WannaCry`.

<img width="741" height="385" alt="image" src="https://github.com/user-attachments/assets/f066008f-ac41-4ed1-b9e7-b6a5b3e6f1a7" />

That's a basic structure of a YARA rule. It starts with a header containing metadata, followed by conditions that define the context of the files to be matched, and a body that specifies the patterns or indicators to be found. The use of metadata and tags helps in organizing and documenting the rules effectively.

## YARA Rule Breakdown

1. **Rule Header:** The rule header provides metadata and identifies the rule. It typically includes:
   - Rule name: A descriptive name for the rule.
   - Rule tags: Optional tags or labels to categorize the rule.
   - Rule metadata: Additional information such as author, description, and creation date.

   **Example:**

<img width="716" height="160" alt="image" src="https://github.com/user-attachments/assets/99746788-4ea8-459e-b708-0e2c751337ce" />

2. **Rule Meta:** The rule meta section allows for the definition of additional metadata for the rule. This metadata can include information about the rule's author, references, version, etc.

   **Example:**

<img width="712" height="278" alt="image" src="https://github.com/user-attachments/assets/71520446-e83d-4511-8fba-9951fd4d55d0" />

3. **Rule Body:** The rule body contains the patterns or indicators to be matched within the files. This is where the actual detection logic is defined.

   **Example:**

<img width="714" height="375" alt="image" src="https://github.com/user-attachments/assets/10ef1046-6737-43f0-9766-06247e80aa9f" />

4. **Rule Conditions:** Rule conditions define the context or characteristics of the files to be matched. Conditions can be based on file properties, strings, or other indicators. Conditions are specified within the condition section.

   **Example:**

<img width="714" height="197" alt="image" src="https://github.com/user-attachments/assets/d2e0a5d4-3d4e-4ae9-8b7b-969c25b6df61" />

In this YARA rule, the condition section simply states all of them, which means that all the strings defined in the rule must be present for the rule to trigger a match.

Let us provide one more example of a condition which specifies that the file size of the analyzed file must be less than 100 kilobytes (KB).

<img width="718" height="99" alt="image" src="https://github.com/user-attachments/assets/b9ec8f93-5fa9-416a-80e4-cde70b9ead52" />

This condition also specifies that the first 2 bytes of the file must be either `0x5A4D` (ASCII MZ) or `0x4D5A` (ASCII ZM), by using `uint16(0)`:

<img width="713" height="72" alt="image" src="https://github.com/user-attachments/assets/de12d553-fd2a-4429-ab27-bd6809581692" />

Here's how `uint16(0)` works:

- **`uint16`:** This indicates the data type to be extracted, which is a 16-bit unsigned integer (2 bytes).
- **`(0)`:** The value inside the parentheses represents the offset from where the extraction should start. In this case, 0 means the function will extract the 16-bit value starting from the beginning of the data being scanned. The condition uses `uint16(0)` to compare the first 2 bytes of the file with specific values.

It's important to note that YARA provides a flexible and extensible syntax, allowing for more advanced features and techniques such as modifiers, logical operators, and external modules. These features can enhance the expressiveness and effectiveness of YARA rules for specific detection scenarios.

Remember that YARA rules can be customized to suit our specific use cases and detection needs. Regular practice and experimentation will further enhance our understanding and proficiency with YARA rule creation. YARA documentation contain more details in depth.

## 🔑 Key Takeaway

YARA scans files and memory for patterns defined in rules, helping SOC analysts detect malware, classify files, identify IOCs, and support incident response and threat hunting. A YARA rule consists of a header, metadata, a body containing patterns, and conditions that determine when a match occurs.
