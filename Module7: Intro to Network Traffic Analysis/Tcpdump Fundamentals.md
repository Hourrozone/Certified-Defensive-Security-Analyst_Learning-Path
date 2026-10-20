# Tcpdump Fundamentals

Tcpdump is a command-line packet sniffer that can directly capture and interpret data frames from a file or network interface. It was built for use on any Unix-like operating system and had a Windows twin called WinDump. It is a potent and straightforward tool used on most Unix-based systems. It does not require a GUI and can be used through any terminal or remote connection, such as SSH. Nevertheless, this tool can seem overwhelming at first due to the many different functions and filters it offers us. However, once we learn the essential functions, we will find it much easier to use this tool efficiently. To capture network traffic from "off the wire," it uses the libraries pcap and libpcap, paired with an interface in promiscuous mode to listen for data. This allows the program to see and capture packets sourcing from or destined for any device in the local area network, not just the packets destined for us.

TCPDump is available for most Unix systems and Unix derivatives, such as AIX, BSD, Linux, Solaris, and is supplied by many manufacturers already in the system. Due to the direct access to the hardware, we need the root or the administrator's privileges to run this tool. For us that means we will have to utilize sudo to execute TCPDump as seen in the examples below. TCPDump often comes preinstalled on the majority of Linux operating systems.

It should be noted that Windows had a port of TCPDump called Windump. Support for windump has ceased. As an alternative running a Linux distribution such as Parrot or Ubuntu in Windows Subsystem for Linux can be an easy way to have a Linux virtual host right on our computer, allowing for the use of TCPDump and many other Linux built tools.

## Locate Tcpdump

To validate if the package exists on our host, use the following command:

```shellsession
[/htb]$ which tcpdump
```

Often it can be found in `/usr/sbin/tcpdump`. However, if the package does not exist, we can install it with:

## Install Tcpdump

```shellsession
[/htb]$ sudo apt install tcpdump
```

We can run the tcpdump package with the `--version` switch to check our install and current package version to validate our install.

## Tcpdump Version Validation

```shellsession
[/htb]$ sudo tcpdump --version

tcpdump version 4.9.3
libpcap version 1.9.1 (with TPACKET_V3)
OpenSSL 1.1.1f  31 Mar 2020
```

## Traffic Captures with Tcpdump

Because of the many different functions and filters, we should first familiarize ourselves with the tool's essential features. Let us discuss some basic TCPDump options, demo some commands, and show how to save traffic to PCAP files and read from these.

### Basic Capture Options

Below is a table of basic Tcpdump switches we can use to modify how our captures run. These switches can be chained together to craft how the tool output is shown to us in STDOUT and what is saved to the capture file. This is not an exhaustive list, and there are many more we can use, but these are the most common and valuable.

| Switch Command | Result |
|---|---|
| `D` | Will display any interfaces available to capture from. |
| `i` | Selects an interface to capture from. ex. `-i eth0` |
| `n` | Do not convert addresses (i.e., host addresses, port numbers, etc.) to names. |
| `e` | Will grab the ethernet header along with upper-layer data. |
| `X` | Show Contents of packets in hex and ASCII. |
| `XX` | Same as X, but will also specify ethernet headers. (like using Xe) |
| `v, vv, vvv` | Increase the verbosity of output shown and saved. |
| `c` | Grab a specific number of packets, then quit the program. |
| `s` | Defines how much of a packet to grab. |
| `S` | change relative sequence numbers in the capture display to absolute sequence numbers. (13248765839 instead of 101) |
| `q` | Print less protocol information. |
| `r file.pcap` | Read from a file. |
| `w file.pcap` | Write into a file |

### Man Page Utilization

To see the complete list of switches, we can utilize the man pages:

#### Tcpdump Man Page

```shellsession
[/htb]$ man tcpdump
```

Here are some examples of basic Tcpdump switch usage along with descriptions of what is happening:

### Listing Available Interfaces

```shellsession
[/htb]$ sudo tcpdump -D
```

<img width="899" height="311" alt="image" src="https://github.com/user-attachments/assets/4a66d95f-acee-427d-a421-7bf650f3438f" />

The above command calls tcpdump using sudo privileges and lists the usable network interfaces. We can choose one of these network interfaces and tell tcpdump which interfaces it should listen to.

### Choosing an Interface to Capture From

```shellsession
[/htb]$ sudo tcpdump -i eth0
```

<img width="872" height="280" alt="image" src="https://github.com/user-attachments/assets/f881001e-8197-4b19-bbf5-b5f8d05e2970" />

In this terminal, we are calling tcpdump and selecting the interface eth0 to capture traffic. Once we issue the command, tcpdump will begin to sniff traffic and see the first few packets across the interface. By issuing the -nn switches as seen below, we tell TCPDump to refrain from resolving IP addresses and port numbers to their hostnames and common port names. In this representation, the last octet is the port from/to which the connection goes.

### Disable Name Resolution

```shellsession
[/htb]$ sudo tcpdump -i eth0 -nn
```

<img width="922" height="553" alt="image" src="https://github.com/user-attachments/assets/04443cef-d525-442d-8b0b-52e468a25214" />

When utilizing the -e switch, we are tasking tcpdump to include the ethernet headers in the capture's output along with its regular content. We can see this worked by examining the output. Usually, the first and second fields consist of the Timestamp and then the IP header's beginning. Now it consists of Timestamp and the source MAC Address of the host.

### Display the Ethernet Header

```shellsession
[/htb]$ sudo tcpdump -i eth0 -e
```
<img width="882" height="456" alt="image" src="https://github.com/user-attachments/assets/b81031d3-8fc4-41f6-8199-9d38405b966e" />


By issuing the -X switch, we can see the packet a bit clearer now. We get an ASCII output on the right to interpret anything in clear text that corresponds to the hexadecimal output on the left.

### Include ASCII and Hex Output

```shellsession
[/htb]$ sudo tcpdump -i eth0 -X
```

<img width="935" height="694" alt="image" src="https://github.com/user-attachments/assets/dd4fddb4-1c02-431a-bb90-2ea7ec8b53ef" />

Pay attention to the level of detail in the output above. We will notice that we have information on the IP header options like time to live, offset, and other flags and more details into the upper layer protocols. Below, we are combining the switches to craft the output to our liking.

### Tcpdump Switch Combinations

```shellsession
[/htb]$ sudo tcpdump -i eth0 -nnvXX
```

<img width="689" height="822" alt="image" src="https://github.com/user-attachments/assets/b2c7bd65-91c3-4574-9189-5f3b3d4b0ced" />

When utilizing the switches, chaining them together as in the example above is best practice.

## Tcpdump Output

When looking at the output from TCPDump, it can be a bit overwhelming. Running through these basic switches has already shown us several different views. We are going to take a minute to dissect that output and explain what we are seeing. The image and table below will define each field. Keep in mind that the more verbose we are with our filters, the more detail from each header is shown.

### Tcpdump Shell Breakdown

<img width="1261" height="629" alt="image" src="https://github.com/user-attachments/assets/c8cdbe9e-cb39-43da-bfbe-5cec00a0ae71" />

| Filter | Result |
|---|---|
| Timestamp | Yellow The timestamp field comes first and is configurable to show the time and date in a format we can ingest easily. |
| Protocol | Orange This section will tell us what the upper-layer header is. In our example, it shows IP. |
| Source & Destination IP.Port | Orange This will show us the source and destination of the packet along with the port number used to connect. Format == IP.port == 172.16.146.2.21 |
| Flags | Green This portion shows any flags utilized. |
| Sequence and Acknowledgement Numbers | Red This section shows the sequence and acknowledgment numbers used to track the TCP segment. Our example is utilizing low numbers to assume that relative sequence and ack numbers are being displayed. |
| Protocol Options | Blue Here, we will see any negotiated TCP values established between the client and server, such as window size, selective acknowledgments, window scale factors, and more. |
| Notes / Next Header | White Misc notes the dissector found will be present here. As the traffic we are looking at is encapsulated, we may see more header information for different protocols. In our example, we can see the TCPDump dissector recognizes FTP traffic within the encapsulation to display it for us. |

There are many other options and information that can be shown. This information varies based on the amount of verbosity that is enabled. For a more detailed understanding of IP and other protocol headers, check out the Networking Primer in section two or the Networking Foundations module.

There is a great advantage in knowing how a network functions and how to use the filters that TCPDump provides. With them, we can view the network traffic, parse it for any issues, and identify suspicious network interactions quickly. Theoretically, we can use tcpdump to create an IDS/IPS system by having a Bash script analyze the intercepted packets according to a specific pattern. We can then set conditions to, for example, ban a particular IP address that has sent too many ICMP echo requests for a certain period.

## File Input/Output with Tcpdump

Using -w will write our capture to a file. Keep in mind that as we capture traffic off the wire, we can quickly use up open disk space and run into storage issues if we are not careful. The larger our network segment, the quicker we will use up storage. Utilizing the switches demonstrated above can help tune the amount of data stored in our PCAPs.

### Save our PCAP Output to a File

```shellsession
[/htb]$ sudo tcpdump -i eth0 -w ~/output.pcap
```
<img width="776" height="149" alt="image" src="https://github.com/user-attachments/assets/e1cfa080-ef6b-4603-ac33-f9b7a0fd1454" />


This capture above will generate the output to a file called output.pcap. When running tcpdump in this way, the output will not scroll our terminal as usual. All output from tcpdump is being redirected to the file we specified for the capture.

### Reading Output From a File

```shellsession
[/htb]$ sudo tcpdump -r ~/output.pcap
```

<img width="684" height="618" alt="image" src="https://github.com/user-attachments/assets/db61bcc4-b8c8-4e73-b21b-63b8b4bb480e" />

This will read the capture stored in output.pcap. Notice it is back to a basic view. To get more detailed information out of the capture file, reapply our switches.

## 🔑 Key Takeaway

Tcpdump is a command-line packet capture and analysis tool that lets us inspect live network traffic or saved PCAP files. Understanding its switches, interface selection, output verbosity, and file input/output options helps us capture relevant traffic, investigate network issues, and identify suspicious network interactions.
