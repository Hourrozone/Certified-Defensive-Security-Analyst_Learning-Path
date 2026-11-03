# Intrusion Detection With Zeek

## Introduction

As already discussed, Zeek, formerly known as Bro, is a powerful network security monitoring tool that allows us to delve deep into our network traffic and extract useful insights.

The flexibility and extensibility of Zeek make it a cornerstone of advanced network-based intrusion detection and investigation. With its rich set of logs and extensive scripting capabilities, we can customize it to suit our specific detection requirements and continuously improve our security posture.

Let's move forward and explore some examples of detecting intrusions with Zeek.

## Intrusion Detection With Zeek Example 1: Detecting Beaconing Malware

Beaconing is a process by which malware communicates with its command and control (C2) server to receive instructions or exfiltrate data. It's usually characterized by a consistent or patterned interval of outbound communications.

By analyzing connection logs (`conn.log`), we can look for patterns in outbound traffic. These patterns can include repetitive connections to the same destination IP or domain, constant data size in the sent data, or the connection timing. These are all indicative of potential beaconing behavior. Anomalies can be further explored using Zeek scripts specifically designed to spot beaconing patterns.

```shell
@htb[/htb]$ /usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/psempire.pcap
```

```shell
@htb[/htb]$ cat conn.log
```

<img width="840" height="854" alt="image" src="https://github.com/user-attachments/assets/46a8d764-1150-4d3f-8449-7560a2038f9e" />

If we look carefully enough we will notice connections being made to `51.15.197.127:80` approximately every 5 seconds, which is indicative of a malware beaconing.

The `psempire.pcap` file, which is located in the `/home/htb-student/pcaps` directory includes traffic related to PowerShell Empire. PowerShell Empire indeed beacons every 5 seconds in its default configuration.

Invest some time in scrutinizing the `psempire.pcap` file using Wireshark.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/psempire.pcap .
```

## Intrusion Detection With Zeek Example 2: Detecting DNS Exfiltration

Zeek is also useful when we suspect data exfiltration. Data exfiltration can be difficult to detect as it often mimics normal network traffic. However, with Zeek, we can analyze our network traffic at a deeper level.

Zeek's `files.log` can be used to identify large amounts of data being sent to an unusual external destination, or over non-standard ports, which may suggest data exfiltration. The `http.log` and `dns.log` can also be utilized to identify potential covert exfiltration channels, such as DNS tunneling or HTTP POST requests to a suspicious domain.

Furthermore, Zeek’s ability to reassemble files transferred over the network (regardless of the protocol used) can assist in identifying the nature of the data being exfiltrated.

**PCAP credits to:** Oleh Levytskyi and Bogdan Vennyk

```shell
@htb[/htb]$ /usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/dnsexfil.pcapng
```

```shell
@htb[/htb]$ cat dns.log
```

<img width="899" height="863" alt="image" src="https://github.com/user-attachments/assets/bc66c87f-296b-43a8-8908-585d66adeb12" />

Let's focus on the requested (sub)domains by leveraging `zeek-cut` as follows.

```shell
@htb[/htb]$ cat dns.log | /usr/local/zeek/bin/zeek-cut query | cut -d . -f1-7
```

<img width="709" height="848" alt="image" src="https://github.com/user-attachments/assets/217516a1-aba2-4ef9-bc59-0e52441db4d7" />

Upon close inspection, it becomes evident that the domain `letsgohunt.online` possesses a significant number of subdomains, similar to cloud providers. However, it's worth noting that interactions with dozens or even hundreds of subdomains are generally not considered typical behavior.

The `dnsexfil.pcapng` file, which is located in the `/home/htb-student/pcaps` directory includes traffic related to DNS exfiltration.

Invest some time in scrutinizing the `dnsexfil.pcapng` file using Wireshark.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/dnsexfil.pcapng .
```

## Intrusion Detection With Zeek Example 3: Detecting TLS Exfiltration

**PCAP credits to:** Oleh Levytskyi and Bogdan Vennyk

Let's now go over an example of detecting data exfiltration over TLS.

```shell
@htb[/htb]$ /usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/tlsexfil.pcap
```

```shell
@htb[/htb]$ cat conn.log
```

<img width="810" height="790" alt="image" src="https://github.com/user-attachments/assets/8864b7d3-32c8-42b3-aadf-566f084ab4da" />

The output is a bit tricky to analyze. Let's narrow things down by using `zeek-cut` one more time.

**One-liner source:** activecountermeasures

```shell
@htb[/htb]$ cat conn.log | /usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes | sort | grep -v -e '^$' | grep -v '-' | datamash -g 1,2 sum 3 | sort -k 3 -rn | head -10
10.0.10.100     192.168.151.181 270775912
10.0.10.100     10.0.10.1       0
```

### Command Breakdown

Let's analyze the command above.

- **`cat conn.log`:** This command is used to read the content of the conn.log file. The conn.log file, generated by Zeek, provides a record of all connections that have taken place in our network.
- **`/usr/local/zeek/bin/zeek-cut id.orig_h id.resp_h orig_bytes`:** In this case, we're extracting the id.orig_h (originating host), id.resp_h (responding host), and orig_bytes (number of bytes sent by the originating host) fields.
- **`sort`:** This command is used to sort the output from the previous command. By default, sort will arrange the lines in ascending order based on the contents of the first field (in this case id.orig_h).
- **`grep -v -e '^$'`:** This command filters out any empty lines. The -v option inverts the selection, the -e option allows for a regular expression, and '^$'` matches empty lines.
- **`grep -v '-'`:** This command filters out lines containing a dash -. In the context of Zeek logs, a dash often represents a missing value or an undefined field.
- **`datamash -g 1,2 sum 3`:** datamash is a command-line tool that performs basic numeric, textual, and statistical operations. The -g 1,2 option groups the output by the first two fields (the IP addresses of the originating and responding hosts), and sum 3 computes the sum of the third field (the number of bytes sent) for each group.
- **`sort -k 3 -rn`:** This command sorts the output of the previous command in descending order (-r) based on the numerical value (-n) of the third field (-k 3), which is the sum of orig_bytes for each pair of IP addresses.
- **`head -10`:** This command is used to limit the output to the top 10 lines, thus showing the top 10 pairs of IP addresses by total bytes sent from the originating host to the responding host.

We notice that ~270 MB (actually a bit less) of data have been sent to `192.168.151.181`.

The `tlsexfil.pcap` file, which is located in the `/home/htb-student/pcaps` directory includes traffic related to data exfiltration over TLS.

Invest some time in scrutinizing the `tlsexfil.pcap` file using Wireshark.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/tlsexfil.pcap .
```

## Intrusion Detection With Zeek Example 4: Detecting PsExec

PsExec, a part of the Sysinternals Suite, is frequently used for remote administration within Active Directory environments. Given its powerful capabilities, it's no surprise that adversaries often prefer PsExec when they carry out remote code execution attacks.

To illustrate a typical attack sequence, let's consider this: an attacker transfers the binary file `PSEXESVC.exe` to a target machine using the `ADMIN$` share, a special shared folder used in Windows networks, via the SMB (Server Message Block) protocol. Following this, the attacker remotely launches this file as a temporary service by utilizing the `IPC$` share, another special shared resource that enables Inter-Process Communication.

We can identify SMB transfers and the typical use of PsExec using Zeek's `smb_files.log`, `dce_rpc.log`, and `smb_mapping.log` as follows.

**PCAP source:** 401TRG

```shell
@htb[/htb]$ /usr/local/zeek/bin/zeek -C -r /home/htb-student/pcaps/psexec_add_user.pcap
```

```shell
@htb[/htb]$ cat smb_files.log
```

<img width="739" height="602" alt="image" src="https://github.com/user-attachments/assets/008ce488-1cf0-4da0-af74-172642a5ef2d" />

```shell
@htb[/htb]$ cat dce_rpc.log
```

<img width="827" height="791" alt="image" src="https://github.com/user-attachments/assets/e4f88aca-2995-408f-93a1-1f3a09ad2f63" />

```shell
@htb[/htb]$ cat smb_mapping.log
```

<img width="716" height="394" alt="image" src="https://github.com/user-attachments/assets/bc50827a-9bd2-4381-a6db-c764a9bb14f0" />

The temporary service creation is apparent in the last two logs above.

The `psexec_add_user.pcap` file, which is located in the `/home/htb-student/pcaps` directory includes traffic related to typical PsExec usage.

Invest some time in scrutinizing the `psexec_add_user.pcap` file using Wireshark.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/psexec_add_user.pcap .
```

## 🔑 Key Takeaway

Zeek can help identify suspicious network behavior by analyzing connection patterns, DNS queries, transferred data, TLS traffic volumes, and SMB activity. Its logs and command-line tools, such as `zeek-cut`, make it possible to investigate beaconing malware, DNS and TLS exfiltration, and remote execution activity such as PsExec.
