# Snort Fundamentals

## Introduction

Snort is an open-source tool, which serves as both an Intrusion Detection System (IDS) and Intrusion Prevention System (IPS), but can also function as a packet logger or sniffer, akin to Suricata. By thoroughly inspecting all network traffic, Snort has the capability to identify and log all activity within that traffic, providing a comprehensive view of the situation and detailed logs of all application layer transactions. We require specific rule sets to instruct Snort on how to perform its inspection and what exactly it needs to identify. Snort was created to operate efficiently on both general-purpose and custom hardware.

## Snort Operation Modes

Snort typically operates in the following modes:

- Inline IDS/IPS
- Passive IDS
- Network-based IDS
- Host-based IDS (however, Snort is not ideally a host-based IDS. We would recommend opting for more specialized tools for this.)

According to Snort's documentation:

> "With certain DAQ modules, Snort is able to utilize two different modes of operation: passive and inline. Passive mode gives Snort the ability to observe and detect traffic on a network interface, but it prevents outright blocking of traffic. Inline mode on the other hand, does give Snort the ability to block traffic if a particular packet warrants such an event.
>
> Snort will infer the particular mode of operation based on the options used at the command line. For example, reading from a pcap file with the -r option or listening on an interface with -i will cause Snort to run in passive mode by default. If the DAQ supports inline, however, then users can specify the -Q flag to run Snort inline.
>
> One DAQ module that supports inline mode is afpacket, which is a module that gives Snort access to packets received on Linux network devices."

## Snort Architecture

In order for Snort to transition from a simple packet sniffer to a robust IDS, several key components were added: Preprocessor, Detection Engine, Logging and Alerting System, and various Output modules.

1. **Packet Sniffer and Packet Decoder**

   The packet sniffer (which includes the Packet Decoder) extracts network traffic, recognizing the structure of each packet. The raw packets that are collected are subsequently forwarded to the Preprocessors.

2. **Preprocessors**

   Preprocessors within Snort identify the type or behaviour of the forwarded packets. Snort has an array of Preprocessor plugins, like the HTTP plugin that distinguishes HTTP-related packets or the port_scan Preprocessor which identifies potential port scanning attempts based on predefined protocols, types of scans, and thresholds. After the Preprocessors have completed their task, information is passed to the Detection Engine. The configuration of these Preprocessors can be found within the Snort configuration file, snort.lua.

3. **Detection Engine**

   The Detection Engine compares each packet with a predefined set of Snort rules. If a match is found, information is forwarded to the Logging and Alerting System.

4. **Logging and Alerting System and Output Modules**

   The Logging and Alerting System and Output modules are in charge of recording or triggering alerts as determined by each rule action. Logs are generally stored in syslog or unified2 formats or directly in a database. The Output modules are configured within the Snort configuration file, snort.lua.

## Snort Configuration & Validating Snort's Configuration

Snort offers a wide range of configuration options, and fortunately, the open-source Snort 3 provides users with pre-configured files to facilitate a quick start. These default configuration files, namely snort.lua and snort_defaults.lua, serve as the foundation for setting up Snort and getting it operational in no time. They provide a standard configuration framework for Snort users.

The snort.lua file serves as the principal configuration file for Snort. This file contains the following sections:

- Network variables
- Decoder configuration
- Base detection engine configuration
- Dynamic library configuration
- Preprocessor configuration
- Output plugin configuration
- Rule set customization
- Preprocessor and decoder rule set customization
- Shared object rule set customization

Let's browse the snort.lua file residing in this section's target as follows.

```shell
@htb[/htb]$ sudo more /root/snorty/etc/snort/snort.lua
```

<img width="653" height="834" alt="image" src="https://github.com/user-attachments/assets/79619457-ac9c-4b2b-bc42-8a118ccd1892" />

Enabling and fine-tuning Snort modules is a significant aspect of the configuration process. To explore the complete list and get a brief description of all Snort 3 modules, you can use the following command.

```shell
@htb[/htb]$ snort --help-modules
```

<img width="756" height="438" alt="image" src="https://github.com/user-attachments/assets/ea09ca6b-13fe-4e35-80db-d9a07b318451" />

These modules are enabled and configured within the snort.lua configuration file as Lua table literals. If a module is initialized as an empty table, it implies that it is utilizing its predefined "default" settings. To view these default settings, you can utilize the following command.

```shell
@htb[/htb]$ snort --help-config arp_spoof
```

<img width="569" height="115" alt="image" src="https://github.com/user-attachments/assets/66064934-46d2-4662-b2e6-d30f4da7abb9" />

Passing (and validating) configuration files to Snort can be done as follows.

```shell
@htb[/htb]$ snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq
```

<img width="591" height="861" alt="image" src="https://github.com/user-attachments/assets/914c3ca3-103a-4a26-a318-b88f67459c55" />

**Note:** --daq-dir /usr/local/lib/daq is not required to pass and validate a configuration file. It is added so that we can replicate the command in this section's target.

Since we mentioned DAQ, Snort 3 should know where to find the appropriate LibDAQ. LibDAQ is the "Data Acquisition Library", and at a high-level, it's an abstraction layer used by modules to communicate with both hardware and software network data sources.

We highly recommend taking the time to read the comments inside the snort.lua file as they provide valuable insights.

## Snort Inputs

To observe Snort in action, the easiest method is to execute it against a packet capture file. By providing the name of the pcap file as an argument to the -r option in the command line, Snort will process the file accordingly.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r /home/htb-student/pcaps/icmp.pcap
```

<img width="642" height="824" alt="image" src="https://github.com/user-attachments/assets/cd7a8f12-41ba-4b69-b4f9-271ee04f7570" />

Snort also has the capability to listen on active network interfaces. To specify this behavior, you can utilize the -i option followed by the names of the interfaces on which Snort should run.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -i ens160
```

<img width="651" height="781" alt="image" src="https://github.com/user-attachments/assets/b07eb0dc-42b8-4e42-aa75-3a8469808d35" />

## Snort Rules

Snort rules, which resemble Suricata rules, are composed of a rule header and rule options. Even though Snort rules share similarities with Suricata rules, we strongly suggest studying Snort rule writing from the following resources: https://docs.snort.org/, https://docs.suricata.io/en/latest/rules/differences-from-snort.html. The most recent Snort rules can be obtained from the Snort website or the Emerging Threats website.

In Snort deployments, we have flexibility in managing rules. It's possible to place rules (for example, local.rules residing at /home/htb-student) directly within the snort.lua configuration file using the ips module as follows.

```shell
@htb[/htb]$ sudo vim /root/snorty/etc/snort/snort.lua
```

<img width="754" height="396" alt="image" src="https://github.com/user-attachments/assets/ea2c97b3-b565-4139-b053-9e97785927f4" />

Then, the "included" rules will be automatically loaded.

In our Snort deployment, we have an alternative approach to incorporate rules directly from the command line. We can pass either a single rules file or a path to a directory containing rules files directly to Snort. This can be achieved using two options:

- **For a single rules file:** we can use the -R option followed by the path to the rules file. This allows us to specify a specific rules file to be utilized by Snort.
- **To include an entire directory of rules files:** we can use the --rule-path option followed by the path to the rules directory. This enables us to provide Snort with a directory containing multiple rules files.

## Snort Outputs

In our Snort deployment, we may encounter a significant amount of data. To provide a summary of the core output types, let's explore the key aspects:

### Basic Statistics

Upon shutdown, Snort generates various counts based on the configuration and processed traffic. This includes:

- **Packet Statistics:** It includes information from the DAQ and decoders, such as the number of received packets and UDP packets.
- **Module Statistics:** Each module keeps track of activity through peg counts, indicating the frequency of observed or performed actions. Examples include the count of processed HTTP GET requests and trimmed TCP reset packets.
- **File Statistics:** This section provides a breakdown of file types, bytes, and signatures.
- **Summary Statistics:** It encompasses the total runtime for packet processing, packets per second, and, if configured, profiling data.

### Alerts

When rules are configured, it is necessary to enable alerting (using the -A option) to view the details of detection events. There are multiple types of alert outputs available, including:

- **`-A cmg`:** This option combines -A fast -d -e and displays alert information along with packet headers and payload.
- **`-A u2`:** This option is equivalent to -A unified2 and logs events and triggering packets in a binary file, which can be used for post-processing with other tools.
- **`-A csv`:** This option outputs fields in comma-separated value format, providing customization options and facilitating pcap analysis.

To discover the available alert types, we can execute the following command.

```shell
@htb[/htb]$ snort --list-plugins | grep logger
```

<img width="541" height="296" alt="image" src="https://github.com/user-attachments/assets/7b166cdd-e252-491c-819f-efa56176999a" />

### Performance Statistics

Beyond the aforementioned outputs, additional data can be obtained. By configuring the perf_monitor module, we can capture a customizable set of peg counts during runtime. This data can be fed into an external program to monitor Snort's activity without interrupting its operation. The profiler module allows tracking of time and space usage by modules and rules. This information is valuable for optimizing system performance. The profiler output appears under Summary Statistics during shutdown.

Let's see an example of the cmg output.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r /home/htb-student/pcaps/icmp.pcap -A cmg
```

<img width="697" height="769" alt="image" src="https://github.com/user-attachments/assets/80b7b0be-94c6-4eab-8382-65ff89b9ae02" />

The same command but using a .rules files that may not be "included" in snort.lua is the following.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -r /home/htb-student/pcaps/icmp.pcap -R /home/htb-student/local.rules -A cmg
```

<img width="749" height="784" alt="image" src="https://github.com/user-attachments/assets/c6ba05a9-08d9-4584-ade5-21d2c8183fd3" />

## Snort Key Features

Key features that bolster Snort's effectiveness include:

- Deep packet inspection, packet capture, and logging
- Intrusion detection
- Network Security Monitoring
- Anomaly detection
- Support for multiple tenants
- Both IPv6 and IPv4 are supported

## 🔑 Key Takeaway

Snort is a flexible network security tool that can operate as an IDS, IPS, packet logger, or sniffer. Its architecture combines packet decoding, preprocessors, a detection engine, and logging and alerting modules to inspect traffic and identify suspicious activity. The `snort.lua` configuration file controls Snort's modules, outputs, and rule customization. Snort can process PCAP files or monitor live network interfaces, load rules through its configuration or command-line options, and produce alerts, statistics, and performance data.
