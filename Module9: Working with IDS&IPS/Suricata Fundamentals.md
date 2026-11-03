# Suricata Fundamentals

Regarded as a potent instrument for Network Intrusion Detection Systems (IDS), Intrusion Prevention Systems (IPS), and Network Security Monitoring (NSM), Suricata represents a cornerstone of network security. This open-source powerhouse, managed and developed by the Open Information Security Foundation (OISF), is a testament to the strength of a community-led, non-profit initiative.

The objective of Suricata is to dissect every iota of network traffic, seeking potential signs of malicious activities. Its strength lies in the ability to not only conduct a sweeping evaluation of our network's condition but also delve into the details of individual application-layer transactions. The key to Suricata's successful operation lies in an intricately designed set of rules. These guidelines direct Suricata's analysis process, identifying potential threats and areas of interest. Equipped to perform at high velocities on both off-the-shelf and specifically designed hardware, Suricata's efficiency is second to none.

## Suricata Operation Modes

Suricata operates in four (4) distinct modes:

1. **Intrusion Detection System (IDS) mode:** The Intrusion Detection System (IDS) mode positions Suricata as a silent observer. In this capacity, Suricata meticulously examines traffic, flagging potential attacks but refraining from any form of intervention. By providing an in-depth view of network activities and accelerating response times, this mode augments network visibility, albeit without offering direct protection.

2. **Intrusion Prevention System (IPS) mode:** In the Intrusion Prevention System (IPS) mode, Suricata adopts a proactive stance. All network traffic must pass through Suricata's stringent checks and is only granted access to the internal network upon Suricata's approval. This mode bolsters security by proactively thwarting attacks before they penetrate our internal network. Deploying Suricata in IPS mode demands an intimate understanding of the network landscape to prevent the inadvertent blocking of legitimate traffic. Furthermore, each rule activation necessitates rigorous testing and validation. While this mode enhances security, the inspection process may introduce latency.

3. **Intrusion Detection Prevention System (IDPS) mode:** The Intrusion Detection Prevention System (IDPS) mode brings together the best of both IDS and IPS. While Suricata continues to passively monitor traffic, it possesses the ability to actively transmit RST packets in response to abnormal activities. This mode strikes a balance between active protection and maintaining low latency, crucial for seamless network operations.

4. **Network Security Monitoring (NSM) mode:** In its Network Security Monitoring (NSM) mode, Suricata transitions into a dedicated logging mechanism, eschewing active or passive traffic analysis or prevention capabilities. It meticulously logs every piece of network information it encounters, providing a valuable wealth of data for retrospective security incident investigations, despite the high volume of data generated.

## Suricata Inputs

Regarding Suricata inputs, there are two main categories:

### Offline Input

- **Offline Input:** This involves reading PCAP files for processing previously captured packets in the LibPCAP file format. It is not only advantageous for conducting post-mortem data examination but also instrumental when experimenting with various rule sets and configurations.

### Live Input

- **Live Input:** Live input can be facilitated via LibPCAP, where packets are read directly from network interfaces. However, LibPCAP is somewhat hamstrung by its performance limitations and lack of load-balancing capabilities. For inline operations, NFQ and AF_PACKET options are available. NFQ, a Linux-specific inline IPS mode, collaborates with IPTables to divert packets from the kernel space into Suricata for detailed scrutiny. Commonly used inline, NFQ necessitates drop rules for Suricata to effectively obstruct packets. Conversely, AF_PACKET provides a performance improvement over LibPCAP and supports multi-threading. Nevertheless, it's not compatible with older Linux distributions and can't be employed inline if the machine is also tasked with routing packets.

Please note that there are other, less commonly used or more advanced inputs available.

## Suricata Outputs

Suricata creates multiple outputs, including logs, alerts, and additional network-related data such as DNS requests and network flows. One of the most critical outputs is EVE, a JSON formatted log that records a wide range of event types including alerts, HTTP, DNS, TLS metadata, drop, SMTP metadata, flow, netflow, and more. Tools such as Logstash can easily consume this output, facilitating data analysis.

We might encounter Unified2 Suricata output, which is essentially a Snort binary alert format, enabling integration with other software that leverages Unified2. Any Unified2 output can be read using Snort’s u2spewfoo tool, which is a straightforward and effective method to gain insight into the alert data.

## Connecting to the Target System

Let's now navigate to the bottom of this section and click on "Click here to spawn the target system!". Then, let's SSH into the Target IP using the provided credentials. The vast majority of the commands covered from this point up to end of this section can be replicated inside the target, offering a more comprehensive grasp of the topics presented.

## Configuring Suricata & Custom Rules

Once we've accessed the deployed Suricata instance over SSH, we can get an overview of all the rule files with a simple execution command.

```shellsession
@htb[/htb]$ ls -lah /etc/suricata/rules/
```

<img width="633" height="797" alt="image" src="https://github.com/user-attachments/assets/2234d868-6485-4b28-8973-4adf3d1679a5" />

The rules can be seen in a straightforward list format and be inspected to understand their functionality, as follows.

```shellsession
@htb[/htb]$ more /etc/suricata/rules/emerging-malware.rules
```

<img width="830" height="735" alt="image" src="https://github.com/user-attachments/assets/571d671e-7c61-4b11-97cb-24f9bfd5e9b6" />

Rules might be commented out, meaning they aren't loaded and don't affect the system. This usually happens when a new version of the rule comes into play or if the threat associated with the rule becomes outdated or irrelevant.

Each rule usually involves specific variables, such as `$HOME_NET` and `$EXTERNAL_NET`. The rule examines traffic from the IP addresses specified in the `$HOME_NET` variable heading towards the IP addresses in the `$EXTERNAL_NET` variable.

These variables can be defined in the suricata.yaml configuration file.

```shellsession
@htb[/htb]$ more /etc/suricata/suricata.yaml
```

<img width="750" height="818" alt="image" src="https://github.com/user-attachments/assets/a133a029-b5c5-4d19-a221-83ade06a5270" />

This allows us to customize these variables according to our specific network environment and even define our own variables.

Finally, to configure Suricata to load signatures from a custom rules file, such as local.rules in the /home/htb-student directory, we would execute the below.

```shellsession
@htb[/htb]$ sudo vim /etc/suricata/suricata.yaml
```

Add /home/htb-student/local.rules to rule-files:

1. Press the Esc key
2. Enter `:wq` and then, press the Enter key

The local.rules file that resides in the /home/htb-student directory of this section's target already contains a Suricata rule. This rule is adequate for this section's learning objectives. We will elaborate more on Suricata rule development in the next section.

## Hands-on With Suricata Inputs

With Suricata inputs, we can experiment with both offline and live input:

### 1. Offline Input

For offline input (reading PCAP files - suspicious.pcap in this case), the following command needs to be executed, and Suricata will create various logs (mainly eve.json, fast.log, and stats.log).

```shellsession
@htb[/htb]$ suricata -r /home/htb-student/pcaps/suspicious.pcap
```

<img width="808" height="250" alt="image" src="https://github.com/user-attachments/assets/c28d05ba-e6d9-4f4b-bda0-b1d4f42c0721" />

An alternative command can be executed to bypass checksums (-k flag) and log in a different directory (-l flag).

```shellsession
@htb[/htb]$ suricata -r /home/htb-student/pcaps/suspicious.pcap -k 
```

<img width="809" height="283" alt="image" src="https://github.com/user-attachments/assets/268e18d5-7f19-4f5a-b55a-29af76d4e1fd" />

### 2. Live Input

For live input, we can try Suricata’s (Live) LibPCAP mode as follows.

```shellsession
@htb[/htb]$ ifconfig
```

<img width="816" height="537" alt="image" src="https://github.com/user-attachments/assets/125e6ffe-75c5-44ba-9ca2-4f5a44d639cf" />

```shellsession
@htb[/htb]$ sudo suricata --pcap=ens160 -vv
[sudo] password for htb-student:
```

<img width="776" height="754" alt="image" src="https://github.com/user-attachments/assets/11e763c0-6cf6-400e-8d3c-09c3a3cdc26a" />

### 3. Inline (NFQ) Mode

For Suricata in Inline (NFQ) mode, the following command should be executed first.

```shellsession
@htb[/htb]$ sudo iptables -I FORWARD -j NFQUEUE
```

Then, we should be able to execute the following.

```shellsession
@htb[/htb]$ sudo suricata -q 0
```

<img width="696" height="168" alt="image" src="https://github.com/user-attachments/assets/49c15458-8b65-4a01-a4d5-0044a2a29262" />

Moreover, to try Suricata in IDS mode with AF_PACKET input, execute one of the below.

```shellsession
@htb[/htb]$ sudo suricata -i ens160
```

<img width="723" height="170" alt="image" src="https://github.com/user-attachments/assets/42f13bc0-4355-4826-97ca-7bd1cb9ed48f" />

```shellsession
@htb[/htb]$ sudo suricata --af-packet=ens160
```

<img width="680" height="155" alt="image" src="https://github.com/user-attachments/assets/c6e3d5f4-73a7-4f8b-94d3-a9447af4d825" />

To observe Suricata dealing with "live" traffic, let's establish an additional SSH connection and utilize tcpreplay to replay network traffic from a PCAP file (suspicious.pcap in this case).

```shellsession
@htb[/htb]$ sudo  tcpreplay -i ens160 /home/htb-student/pcaps/suspicious.pcap
```

<img width="800" height="264" alt="image" src="https://github.com/user-attachments/assets/1219f866-2128-438d-a27f-39dca0a35e65" />

Then, feel free to terminate both tcpreplay and Suricata. The logs from the observed (replayed) traffic will be available at /var/log/suricata

The -i option helps Suricata choose the best input option. In the case of Linux, the best input option is AF_PACKET. If pcap mode is needed, the --pcap option is recommended. Configuration of (Live) LibPCAP can be achieved via the suricata.yaml file, including settings for buffer size, BPF or tcpdump filters, checksum validation, threads, promiscuous mode, snap length, etc.

## Hands-on With Suricata Outputs

Suricata records a variety of data into logs that reside in the /var/log/suricata directory by default. For us to access and manipulate these logs, we require root-level access. Among these logs, we find the eve.json, fast.log, and stats.log files, which provide invaluable insight into the network activity. Let's delve into each:

### 1. eve.json

This file is Suricata's recommended output and contains JSON objects, each carrying diverse information such as timestamps, flow_id, event_type, and more. Try inspecting the content of old_eve.json residing at /var/log/suricata as follows.

```shellsession
@htb[/htb]$ less /var/log/suricata/old_eve.json
```

<img width="791" height="784" alt="image" src="https://github.com/user-attachments/assets/a0d6d079-9758-4f64-a0ce-988ecc4ed375" />

If we wish to filter out only alert events, for example, we can utilize the jq command-line JSON processor as follows.

```shellsession
@htb[/htb]$ cat /var/log/suricata/old_eve.json | jq -c 'select(.event_type == "alert")'
```

<img width="713" height="317" alt="image" src="https://github.com/user-attachments/assets/592517da-eb7c-4f50-a8dd-ba9d483e48cb" />

If we wish to identify the earliest DNS event, for example, we can utilize the jq command-line JSON processor as follows.

```shellsession
@htb[/htb]$ cat /var/log/suricata/old_eve.json | jq -c 'select(.event_type == "dns")' | head -1 | jq .
```

<img width="640" height="535" alt="image" src="https://github.com/user-attachments/assets/1178f7e8-a74e-47df-8f9a-9d4dd24dfa6a" />

We can also use similar commands to filter for other event types like TLS and SSH.

**flow_id:** This is a unique identifier assigned by Suricata to each network flow. A flow, in Suricata terms, is defined as a set of IP packets passing through a network interface in a specific direction and between a given pair of source and destination endpoints. Each of these flows gets a unique flow_id. This identifier helps us track and correlate various events related to the same network flow in the EVE JSON log. Using flow_id, we can associate different pieces of information related to the same flow, such as alerts, network transactions, and packets, providing a cohesive view of what is happening on a specific communication channel.

**pcap_cnt:** This is a counter that Suricata increments for each packet it processes from the network traffic or from a PCAP file (in offline mode). pcap_cnt allows us to trace a packet back to its original order in the PCAP file or network stream. This is beneficial in understanding the sequence of network events as they occurred. It can help to precisely pinpoint when an alert was triggered in relation to other packets, which can provide valuable context in an investigation.

### 2. fast.log

This is a text-based log format that records alerts only and is enabled by default. Try inspecting the content of old_fast.log residing at /var/log/suricata as follows.

```shellsession
@htb[/htb]$ cat /var/log/suricata/old_fast.log
```

<img width="670" height="150" alt="image" src="https://github.com/user-attachments/assets/b371a455-aaba-4a60-b5fc-175cfe0c0196" />

### 3. stats.log

This is a human-readable statistics log, which can be particularly useful while debugging Suricata deployments. Try inspecting the content of old_stats.log residing at /var/log/suricata as follows.

```shellsession
@htb[/htb]$ cat /var/log/suricata/old_stats.log
```

<img width="741" height="783" alt="image" src="https://github.com/user-attachments/assets/328026d9-203b-4018-96cb-3285269ec22b" />

For those of us who want a more focused output strategy, there's an option to deactivate the comprehensive EVE output and activate particular outputs instead. Take http-log, for instance. By activating this, every time Suricata is run and encounters HTTP events, a fresh http.log file will be generated.

## Hands-on With Suricata Outputs - File Extraction

Suricata, has an underused yet highly potent feature - file extraction. This feature allows us to capture and store files transferred over a number of different protocols, providing invaluable data for threat hunting, forensics, or simply data analysis.

Here's how we go about enabling file extraction in Suricata.

We start by making changes to the Suricata configuration file (suricata.yaml). In this file, we'll find a section named file-store. This is where we tell Suricata how to handle the files it extracts. Specifically, we need to set version to 2, enabled to yes, and the force-filestore option also to yes. The resulting section should look something like this.

```yaml
file-store:
  version: 2
  enabled: yes
  force-filestore: yes
```

In the same file-store section, we define where Suricata stores the extracted files. We set the dir option to the directory of our choice.

As a quick exercise, let's enable file extraction and run Suricata on the /home/htb-student/pcaps/vm-2.pcap file from www.netresec.com.

In accordance with the guidelines put forth in Suricata's documentation, file extraction isn't an automatic process that occurs without our explicit instructions. It's fundamentally crucial for us to craft a specific rule that instructs Suricata when and what kind of files it should extract.

The simplest rule we can add to our local.rules file to experiment with file extraction is the following.

```text
alert http any any -> any any (msg:"FILE store all"; filestore; sid:2; rev:1;)
```

If we configured Suricata correctly, multiple files will be stored inside the filestore directory.

Let's run Suricata on the /home/htb-student/pcaps/vm-2.pcap file.

```shellsession
@htb[/htb]$ suricata -r /home/htb-student/pcaps/vm-2.pcap
```

<img width="735" height="193" alt="image" src="https://github.com/user-attachments/assets/45ee12cb-0e8e-4519-a23c-8112f1a7ae58" />

We will notice that eve.json, fast.log, stats.log, and suricata.log were created, alongside a new directory called filestore. filestore's content in terms of the files it contains can be inspected as follows.

```shellsession
@htb[/htb]$ cd filestore
@htb[/htb]$ find . -type f
```

<img width="595" height="443" alt="image" src="https://github.com/user-attachments/assets/e8c130ad-e944-458f-8ce9-ec895ce456b2" />

Again in accordance with the guidelines put forth in Suricata's documentation the file-store module uses its own log directory (default: filestore in the default logging directory) and logs files using the SHA256 of the contents as the filename. Each file is then placed in a directory named 00 to ff where the directory shares the first 2 characters of the filename. For example, if the SHA256 hex string of an extracted file starts with f9bc6d... the file we be placed in the directory filestore/f9.

If we wanted to inspect, for example, the /21/21742fc621f83041db2e47b0899f5aea6caa00a4b67dbff0aae823e6817c5433 file inside the filestore directory, we could use the xxd tool as follows.

```shellsession
@htb[/htb]$ cd filestore
@htb[/htb]$ xxd 
```

<img width="640" height="311" alt="image" src="https://github.com/user-attachments/assets/533f891d-5245-4120-99c8-f5f0ade1fb80" />

In this case, the file was a Windows executable based on the file's header. More about the MS-DOS EXE format can be found in following resource MZ.

## Live Rule Reloading Feature & Updating Suricata Rulesets

Live rule reloading is a crucial feature in Suricata that allows us to update our ruleset without interrupting ongoing traffic inspection. This feature provides continuous monitoring and minimizes the chances of missing any malicious activity.

To enable live rule reloading in Suricata, we need to configure our Suricata configuration file (suricata.yaml). In the suricata.yaml file, we should locate the detect-engine section and set the value of the reload parameter to true. It looks something like this:

```yaml
detect-engine:
  - reload: true
```

Proceed to execute the following kill command, which will signal the Suricata process (determined by `$(pidof suricata)`) to refresh its rule set without necessitating a complete restart.

```shellsession
@htb[/htb]$ sudo kill -usr2 $(pidof suricata)
```

This modification tells Suricata to check for changes in the ruleset periodically and apply them without needing to restart the service.

Most of the commands below cannot be replicated inside this section's target since they require internet connectivity.

<img width="787" height="779" alt="image" src="https://github.com/user-attachments/assets/6997cf82-e07d-4f97-9e5d-ec18dcbbf195" />


### Updating Suricata's Ruleset

Updating Suricata's ruleset can be performed using the suricata-update tool. We can perform a simple update to the Suricata ruleset using the following command.

```shellsession
@htb[/htb]$ sudo suricata-update
```

(image here)

As displayed in the example above, the output indicates that the suricata-update command has successfully retrieved the rules by establishing a connection with https://rules.emergingthreats.net/open/. Subsequently, the command saves the newly obtained rules to the /var/lib/suricata/rules/ directory.

Moving forward, let's execute the command provided below to generate a comprehensive list of all ruleset providers.

```shellsession
@htb[/htb]$ sudo suricata-update list-sources
```

<img width="760" height="803" alt="image" src="https://github.com/user-attachments/assets/e87d5e87-e480-4727-a065-1c938948d500" />

Let's make a note from the output above of a specific ruleset name from which we want Suricata to fetch rulesets. The et/open rulesets serve as an excellent choice for demonstration purposes.

Next, let's proceed with executing the following command to retrieve and enable the et/open rulesets within our Suricata rules.

```shellsession
@htb[/htb]$ sudo suricata-update enable-source et/open
```

<img width="763" height="190" alt="image" src="https://github.com/user-attachments/assets/ebafd4f0-abcb-44ef-b801-8f41f9547215" />

Lastly, let's reissue the suricata-update command to load the newly acquired ruleset.

```shellsession
@htb[/htb]$ sudo suricata-update
```

A Suricata service restart may also be required.

```shellsession
@htb[/htb]$ sudo systemctl restart suricata
```

## Validating Suricata's Configuration

Validation of Suricata's configuration is also an essential part of maintaining the robustness of our IDS/IPS setup. To validate the configuration, we can use the -T option provided by the Suricata command. This command runs a test to check if the configuration file is valid and all files referenced in the configuration are accessible.

Here is how we can do this.

```shellsession
@htb[/htb]$ sudo suricata -T -c /etc/suricata/suricata.yaml
```

<img width="726" height="127" alt="image" src="https://github.com/user-attachments/assets/1f7aa42b-a216-4a38-b150-6b591844dec8" />

## Suricata Documentation

Suricata is an incredibly versatile tool with extensive functionality. We highly recommend exploring Suricata's documentation to gain a deeper understanding of its capabilities.

## Suricata Key Features

Key features that bolster Suricata's effectiveness include:

- Deep packet inspection and packet capture logging
- Anomaly detection and Network Security Monitoring
- Intrusion Detection and Prevention, with a hybrid mode available
- Lua scripting
- Geographic IP identification (GeoIP)
- Full IPv4 and IPv6 support
- IP reputation
- File extraction
- Advanced protocol inspection
- Multitenancy

**Note:** Suricata can also be used to detect "non-standard/anomalous" traffic. We can leverage strategies outlined in Suricata's Protocol Anomalies Detection page. This approach enhances our visibility into unusual or non-compliant behavior within our network, thus augmenting our security posture.

## 🔑 Key Takeaway

Suricata is a versatile network security tool that supports IDS, IPS, IDPS, and NSM operation modes. It can process offline PCAP files or inspect live network traffic through different input methods, including LibPCAP, NFQ, and AF_PACKET. Its outputs—especially `eve.json`, `fast.log`, and `stats.log`—provide valuable information for monitoring and investigation. Additional capabilities include file extraction, live rule reloading, ruleset updates, configuration validation, and detection of anomalous network traffic.
