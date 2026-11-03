# Snort Rule Development

## Introduction

A Snort rule, at its core, is a powerful tool that we use to identify and flag potential malicious activity in network traffic.

As already discussed, Snort rules resemble Suricata rules. They are composed of a rule header and rule options. Even though Snort rules share similarities with Suricata rules, we strongly suggest studying Snort rule writing from the following resources: https://docs.snort.org/, https://docs.suricata.io/en/latest/rules/differences-from-snort.html.

## Snort Rule Development Example 1: Detecting Ursnif (Inefficiently)

```text
alert tcp any any -> any any (msg:"Possible Ursnif C2 Activity"; flow:established,to_server; content:"/images/", depth 12; content:"_2F"; content:"_2B"; content:"User-Agent|3a 20|Mozilla/4.0 (compatible|3b| MSIE 8.0|3b| Windows NT"; content:!"Accept"; content:!"Cookie|3a|"; content:!"Referer|3a|"; sid:1000002; rev:1;)
```

The Snort rule above is designed to detect certain variations of Ursnif malware. The rule is inefficient since it misses HTTP sticky buffers. Let's break down the important parts of this rule to understand its workings.

- **`flow:established,to_server;`** ensures that this rule only matches established TCP connections where data is flowing from the client to the server.
- **`content:"/images/", depth 12;`** instructs Snort to look for the string /images/ within the first 12 bytes of the packet payload.
- **`content:"_2F";` and `content:"_2B";`** direct Snort to search for the strings _2F and _2B anywhere in the payload.
- **`content:"User-Agent|3a 20|Mozilla/4.0 (compatible|3b| MSIE 8.0|3b| Windows NT";`** is looking for a specific User-Agent. The |3a 20| and |3b| in the rule are hexadecimal representations of the : and ; characters respectively.
- **`content:!"Accept"; content:!"Cookie|3a|"; content:!"Referer|3a|";`** look for the absence of certain standard HTTP headers, such as Accept, Cookie: and Referer:. The ! indicates negation.

The above rule is already incorporated in the local.rules file found in the /home/htb-student directory of this section's target. To test it, first, you need to uncomment the rule. Then, execute Snort on the ursnif.pcap file, which is located in the /home/htb-student/pcaps directory.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R /home/htb-student/local.rules -r /home/htb-student/pcaps/ursnif.pcap -A cmg
```

<img width="761" height="822" alt="image" src="https://github.com/user-attachments/assets/60770bfc-f3c0-4994-ad8e-0292ceb6b903" />

Invest some time in scrutinizing both the ursnif.pcap file using Wireshark and this rule to comprehend how it works.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/ursnif.pcap .
```

## Snort Rule Development Example 2: Detecting Cerber

```text
alert udp $HOME_NET any -> $EXTERNAL_NET any (msg:"Possible Cerber Check-in"; dsize:9; content:"hi", depth 2, fast_pattern; pcre:"/^[af0-9]{7}$/R"; detection_filter:track by_src, count 1, seconds 60; sid:2816763; rev:4;)
```

The Snort rule above is designed to detect certain variations of Cerber malware. Let's break down the important parts of this rule to understand its workings.

- **`$HOME_NET any -> $EXTERNAL_NET any`** signifies the rule applies to any UDP traffic going from any port in the home network to any port on external networks.
- **`dsize:9;`**: This is a condition that restricts the rule to UDP datagrams that have a payload data size of exactly 9 bytes.
- **`content:"hi", depth 2, fast_pattern;`**: This checks the payload's first 2 bytes for the string hi. The fast_pattern modifier makes the pattern matcher search for this pattern before any others in the rule, optimizing the rule's performance.
- **`pcre:"/^[af0-9]{7}$/R";`**: This stands for Perl Compatible Regular Expressions. The rule is looking for seven hexadecimal characters (from the set a-f and 0-9) starting at the beginning of the payload (after the hi), and this should be the complete payload (signified by the $ end anchor).
- **`detection_filter:track by_src, count 1, seconds 60;`**: The detection_filter keyword in Snort rule language is used to suppress alerts unless a certain threshold of matched events occurs within a specified time frame. In this rule, the filter is set to track by source IP (by_src), with a count of 1 and within a time frame of 60 seconds. This means that the rule will trigger an alert only if it matches more than one event (specifically, more than count events which is 1 here) from the same source IP address within 60 seconds.

The above rule is already incorporated in the local.rules file found in the /home/htb-student directory of this section's target. To test it, first, you need to uncomment the rule. Then, execute Snort on the cerber.pcap file, which is located in the /home/htb-student/pcaps directory.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R /home/htb-student/local.rules -r /home/htb-student/pcaps/cerber.pcap -A cmg
```

<img width="778" height="815" alt="image" src="https://github.com/user-attachments/assets/d48a23c8-b63a-4fef-88de-54c42b0674b7" />

Invest some time in scrutinizing both the cerber.pcap file using Wireshark and this rule to comprehend how it works.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/cerber.pcap .
```

## Snort Rule Development Example 3: Detecting Patchwork

```text
alert http $HOME_NET any -> $EXTERNAL_NET any (msg:"OISF TROJAN Targeted AutoIt FileStealer/Downloader CnC Beacon"; flow:established,to_server; http_method; content:"POST"; http_uri; content:".php?profile="; http_client_body; content:"ddager=", depth 7; http_client_body; content:"&r1=", distance 0; http_header; content:!"Accept"; http_header; content:!"Referer|3a|"; sid:10000006; rev:1;)
```

The Snort rule above is designed to detect certain variations of malware used by the Patchwork APT. Please notice the use of HTTP sticky buffers in this rule. Let's break down the important parts of this rule to understand its workings.

- **`flow:established,to_server;`**: This keyword is used to specify the direction of the traffic we are interested in. In this case, we're looking at established connections where the traffic is going from the client to the server.
- **`http_method; content:"POST";`**: We are looking for HTTP traffic where the method used is POST.
- **`http_uri; content:".php?profile=";`**: This specifies that we're looking for HTTP URIs that contain the string .php?profile=.
- **`http_client_body; content:"ddager=", depth 7;`**: We're examining the body of the HTTP request. Specifically, we're looking for the string ddager= within the first 7 bytes of the body.
- **`http_client_body; content:"&r1=", distance 0;`**: We're still examining the body of the HTTP request, but now we're looking for the string &r1= immediately following the previous content match.
- **`http_header; content:!"Accept"; http_header; content:!"Referer|3a|";`**: These conditions are looking for the absence of the Accept and Referer HTTP headers. The ! before the content means not, so we're looking for situations where these headers are not present.

The above rule is already incorporated in the local.rules file found in the /home/htb-student directory of this section's target. To test it, first, you need to uncomment the rule. Then, execute Snort on the patchwork.pcap file, which is located in the /home/htb-student/pcaps directory.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R /home/htb-student/local.rules -r /home/htb-student/pcaps/patchwork.pcap -A cmg
```

<img width="748" height="789" alt="image" src="https://github.com/user-attachments/assets/966deb00-c9a7-4d60-a461-c43e47040942" />

Invest some time in scrutinizing both the patchwork.pcap file using Wireshark and this rule to comprehend how it works.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/patchwork.pcap .
```

## Snort Rule Development Example 4: Detecting Patchwork (SSL)

```text
alert tcp $EXTERNAL_NET any -> $HOME_NET any (msg:"Patchwork SSL Cert Detected"; flow:established,from_server; content:"|55 04 03|"; content:"|08|toigetgf", distance 1, within 9; classtype:trojan-activity; sid:10000008; rev:1;)
```

The Snort rule above is designed to detect certain variations of malware used by the Patchwork APT. Let's break down the important parts of this rule to understand its workings.

- **`flow:established,from_server;`**: This keyword pair signifies that we're interested in observing established flows of traffic originating from the server.
- **`content:"|55 04 03|";`**: This rule is looking for the specific hex values 55 04 03 within the payload of the packet. These hex values represent the ASN.1 (Abstract Syntax Notation One) tag for the "common name" field in an X.509 certificate, which is often used in SSL/TLS certificates to denote the domain name that the certificate applies to.
- **`content:"|08|toigetgf", distance 1, within 9;`**: Following the common name field, this rule looks for the string toigetgf. The distance 1 means that Snort should start looking for the string toigetgf 1 byte after the end of the previous content match. The within 9 sets an upper limit on how far into the packet's payload Snort should search, starting from the beginning of this content field.

The above rule is already incorporated in the local.rules file found in the /home/htb-student directory of this section's target. To test it, first, you need to uncomment the rule. Then, execute Snort on the patchwork.pcap file, which is located in the /home/htb-student/pcaps directory.

```shell
@htb[/htb]$ sudo snort -c /root/snorty/etc/snort/snort.lua --daq-dir /usr/local/lib/daq -R /home/htb-student/local.rules -r /home/htb-student/pcaps/patchwork.pcap -A cmg
```

<img width="746" height="820" alt="image" src="https://github.com/user-attachments/assets/1c105660-0628-4d79-9ca4-4be7d66aa462" />

Invest some time in scrutinizing both the patchwork.pcap file using Wireshark and this rule to comprehend how it works.

We can download the PCAP file into the current directory of either Pwnbox or our own VM as follows.

```shell
@htb[/htb]$ scp htb-student@[TARGET IP]:/home/htb-student/pcaps/patchwork.pcap .
```

## 🔑 Key Takeaway

Snort rules combine a header, which defines the traffic to inspect, with options that identify specific patterns and conditions. These examples demonstrate how rules can detect malware activity through TCP and UDP traffic characteristics, HTTP methods, URIs, request bodies, missing headers, and SSL certificate fields. Effective rule development requires understanding traffic direction, content matching, sticky buffers, regular expressions, and detection filters. Testing rules against PCAP files with Snort and examining the traffic in Wireshark helps validate and refine detection logic.
