# Tcpdump Packet Filtering

Tcpdump provides a robust and efficient way to parse the data included in our captures via packet filters. This section will examine those filters and get a glimpse at how it modifies the output from our capture.

## Filtering and Advanced Syntax Options

Utilizing more advanced filtering options like those listed below will enable us to trim down what traffic is printed to output or sent to file. By reducing the amount of info we capture and write to disk, we can help reduce the space needed to write the file and help the buffer process data quicker. Filters can be handy when paired with standard tcpdump syntax options. We can capture as widely as we wish, or be super specific only to capture packets from a particular host, or even with a particular bit in the TCP header set to on. It is highly recommended to explore the more advanced filters and find different combinations.

These filters and advanced operators are by no means an exhaustive list. They were chosen because they are the most frequently used and will get us up and running quickly. When implemented, these filters will inspect any packets captured and look for the given values in the protocol header to match.

### Helpful TCPDump Filters

| Filter | Result |
|---|---|
| host | host will filter visible traffic to show anything involving the designated host. Bi-directional |
| src / dest | src and dest are modifiers. We can use them to designate a source or destination host or port. |
| net | net will show us any traffic sourcing from or destined to the network designated. It uses / notation. |
| proto | will filter for a specific protocol type. (ether, TCP, UDP, and ICMP as examples) |
| port | port is bi-directional. It will show any traffic with the specified port as the source or destination. |
| portrange | portrange allows us to specify a range of ports. (0-1024) |
| less / greater "< >" | less and greater can be used to look for a packet or protocol option of a specific size. |
| and / && | and && can be used to concatenate two different filters together. for example, src host AND port. |
| or | or allows for a match on either of two conditions. It does not have to meet both. It can be tricky. |
| not | not is a modifier saying anything but x. For example, not UDP. |

With these filters, we can filter the network traffic on most properties to facilitate the analysis. Let us look at some examples of these filters and how they look when we use them. When using the host filter, whatever IP we input will be checked for in the source or destination IP field. This can be seen in the output below.

### Host Filter

```shellsession
@htb[/htb]$ ### Syntax: host [IP]
@htb[/htb]$ sudo tcpdump -i eth0 host 172.16.146.2
```

<img width="726" height="387" alt="image" src="https://github.com/user-attachments/assets/d0026881-b88d-42d2-aa76-e2c786bd9675" />

This filter is often used when we want to examine only a specific host or server. With this, we can identify with whom this host or server communicates and in which way. Based on our network configurations, we will understand if this connection is legitimate. If the communication seems strange, we can use other filters and options to view the content in more detail. Besides the individual hosts, we can also define the source host as well as the target host. We can also define entire networks and their ports.

### Source/Destination Filter

```shellsession
@htb[/htb]$ ### Syntax: src/dst [host|net|port] [IP|Network Range|Port]
@htb[/htb]$ sudo tcpdump -i eth0 src host 172.16.146.2
```

<img width="730" height="653" alt="image" src="https://github.com/user-attachments/assets/2847018e-9bf1-45bd-ae20-4b0f23bdb7ba" />

Source and destination allow us to work with the directions of communication. For example, in the last output, we have specified that our source host is 172.16.146.2, and only packets sent from this host will be intercepted. This can be done for ports, and network ranges as well. An example of this utilizing src port # would look something like this:

### Utilizing Source With Port as a Filter

```shellsession
@htb[/htb]$ sudo tcpdump -i eth0 tcp src port 80
```

<img width="743" height="382" alt="image" src="https://github.com/user-attachments/assets/d48cdd15-28a4-4d86-ad20-3e3f86b65f51" />

Notice now that we only see one side of the conversation? This is because we are filtering on the source port of 80 (HTTP). Used in this manner, net will grab anything matching the / notation for a network. In the example, we are looking for anything destined for the 172.16.146.0/24 network.

### Using Destination in Combination with the Net Filter

```shellsession
@htb[/htb]$ sudo tcpdump -i eth0 dest net 172.16.146.0/24
```

<img width="733" height="527" alt="image" src="https://github.com/user-attachments/assets/af5a5743-034b-4822-8c28-5b6ccdea13c6" />

This filter can utilize the common protocol name or protocol number for any IP, IPv6, or Ethernet protocol. Common examples would be tcp[6], udp[17], or icmp[1]. In the outputs below, we will utilize both the common name (top) and the protocol number (bottom). We can see it produced the same output. For the most part, these are interchangeable, but utilizing proto will become more useful when you are starting to dissect a specific part of the IP or other protocol headers. It will be more apparent later in this section when we talk about looking for TCP flags. We can take a look at this resource for a helpful list covering protocol numbers.

### Protocol Filter - Common Name

```shellsession
@htb[/htb]$ ### Syntax: [tcp/udp/icmp]
@htb[/htb]$ sudo tcpdump -i eth0 udp
```

<img width="721" height="199" alt="image" src="https://github.com/user-attachments/assets/38d68096-14f1-4dc5-8d10-e6bcf66094cc" />

### Protocol Filter - Number

```shellsession
@htb[/htb]$ ### Syntax: proto [protocol number]
@htb[/htb]$ sudo tcpdump -i eth0 proto 17
```

<img width="733" height="202" alt="image" src="https://github.com/user-attachments/assets/d78ba8a3-b979-4ee6-9ed9-1aeccfc586ee" />

Using the port filter, we should keep in mind what we are looking for and how that protocol functions. Some standard protocols like HTTP or HTTPS only use ports 80 and 443 with the transport protocol of TCP. With that in mind, picture ports as a simple way to establish connections and protocols like TCP and UDP to determine if they use an established method. Ports by themselves can be used for anything, so filtering on port 80 will show all traffic over that port number. However, if we are looking to capture all HTTP traffic, utilizing tcp port 80 will ensure we only see HTTP traffic.

With protocols that use both TCP and UDP for different functions, such as DNS, we can filter looking at one or the other TCP/UDP port 53 or filter for port 53. By doing this, we will see any traffic-utilizing that port, regardless of the transport protocol.

### Port Filter

```shellsession
@htb[/htb]$ ### Syntax: port [port number]
@htb[/htb]$ sudo tcpdump -i eth0 tcp port 443
```

<img width="653" height="734" alt="image" src="https://github.com/user-attachments/assets/2e740ce3-c20d-4fd6-9477-6100572d317b" />

Apart from the individual ports, we can also define specific ranges of these ports, which are then listened to by TCPdump. Listening on a range of ports can be especially useful when we see network traffic from ports that do not match the services running on our servers. For example, if we have a web server with TCP ports 80 and 443 running in a particular segment of our network and suddenly have outgoing network traffic from TCP port 10000 or others, it is very suspicious.

The portrange filter, as seen below, allows us to see everything from within the port range. In the example, we see some DNS traffic along with some HTTP web requests.

### Port Range Filter

```shellsession
@htb[/htb]$ ### Syntax: portrange [portrange 0-65535]
@htb[/htb]$ sudo tcpdump -i eth0 portrange 0-1024
```

<img width="641" height="516" alt="image" src="https://github.com/user-attachments/assets/e3e91945-01d2-4eb1-97be-73868aefdf6c" />

Next, we are looking for any packet less than 64 bytes. From the following output, we can see that for this capture, those packets mainly consisted of SYN, FIN, or KeepAlive packets. Less than and greater than can be a helpful modifier set. For example, let us say we are looking to capture traffic that includes a file transfer or set of files. We know these files will be larger than regular traffic. To demonstrate, we can utilize greater 500 (alternatively '>500'), which will only show us packets with a size larger than 500 bytes. This will strip out all the extra packets from the view we know we are not concerned with already.

### Less/Greater Filter

```shellsession
@htb[/htb]$ ### Syntax: less/greater [size in bytes]
@htb[/htb]$ sudo tcpdump -i eth0 less 64
```

<img width="670" height="766" alt="image" src="https://github.com/user-attachments/assets/6c159f7b-b6ec-4d72-a471-73d0af943e60" />

Above was an excellent example of using less. We can utilize the modifier greater 500 to only show me packets with 500 or more bytes. It came back with a unique response in the ASCII. Can we tell what happened here?

### Utilizing Greater

```shellsession
@htb[/htb]$ sudo tcpdump -i eth0 greater 500
```

<img width="629" height="329" alt="image" src="https://github.com/user-attachments/assets/6488fc62-5880-4e2a-802d-bcb0c402dabe" />

AND as a modifier will show us anything that meets both requirements set. For example, host 10.12.1.122 and tcp port 80 will look for anything from the source host and contain port 80 TCP, or we can filter for UDP traffic on a specific port. Both criteria have to be met for the filter to capture the packet. We can see this in action below. Here we utilize host 192.168.0.1 and port 23 as a filter. So we will see only traffic that is from this particular host that is only port 23 traffic.

### AND Filter

```shellsession
@htb[/htb]$ ### Syntax: and [requirement]
@htb[/htb]$ sudo tcpdump -i eth0 host 192.168.0.1 and port 23
```

<img width="644" height="327" alt="image" src="https://github.com/user-attachments/assets/4dd32faa-9dda-45d8-81cf-d003489beb6f" />

The other modifiers, OR and NOT provide us with a way to specify multiple conditions or negate something. Let us play with that a bit now. What do we notice about this output?

### Basic Capture With No Filter

```shellsession
@htb[/htb]$ sudo tcpdump -i eth0
```

<img width="657" height="605" alt="image" src="https://github.com/user-attachments/assets/0e153ec1-8cda-45b8-b10d-8d24d9f2b917" />

We have a mix of different sources and destinations along with multiple protocol types. If we were to use the OR (alternatively ||) modifier, we could ask for traffic from a specific host or just ICMP traffic as an example. Let us rerun it and add in an OR.

### OR Filter

```shellsession
@htb[/htb]$ ### Syntax: or/|| [requirement]
@htb[/htb]$ sudo tcpdump -r sus.pcap icmp or host 172.16.146.1
```

<img width="655" height="580" alt="image" src="https://github.com/user-attachments/assets/6d5213ff-9b0c-464f-a016-60c0679930a9" />

Our traffic looks a bit different now. That is because a lot of the packets matched the ICMP variable while some matched the host variable. So in this output, we can see some ARP traffic and ICMP traffic. The filter worked since 172.16.146.2 matched the other variable and appeared as a host in either the source or destination field. Now, what happens if we utilize the NOT (alternatively !) modifier.

### NOT Filter

```shellsession
@htb[/htb]$ ### Syntax: not/! [requirement]
@htb[/htb]$ sudo tcpdump -r sus.pcap not icmp
```

<img width="650" height="617" alt="image" src="https://github.com/user-attachments/assets/444e6718-ae2f-4a28-b7a3-f27c6c0b8ff9" />

It looks much different now. We only see some ARP traffic, and then we see some HTTPS traffic we did not get to before. This is because we negated any ICMP traffic from being displayed using not icmp.

## Pre-Capture Filters VS. Post-Capture Processing

When utilizing filters, we can apply them directly to the capture or apply them when reading a capture file. By applying them to the capture, it will drop any traffic not matching the filter. This will reduce the amount of data in the captures and potentially clear out traffic we may need later, so use them only when looking for something specific, such as troubleshooting a network connectivity issue. When applying the filter to capture, we have read from a file, and the filter will parse the file and remove anything from our terminal output not matching the specified filter. Using a filter in this way can help us investigate while saving potential valuable data in the captures. It will not permanently change the capture file, and to change or clear the filter from our output will require we rerunning our command with a change in the syntax.

## Interpreting Tips and Tricks

Using the -S switch will display absolute sequence numbers, which can be extremely long. Typically, tcpdump displays relative sequence numbers, which are easier to track and read. However, if we look for these values in another tool or log, we will only find the packet based on absolute sequence numbers. For example, 13245768092588 to 100.

The -v, -X, and -e switches can help you increase the amount of data captured, while the -c, -n, -s, -S, and -q switches can help reduce and modify the amount of data written and seen.

Many handy options that can be used but are not always directly valuable for everyone are the -A and -l switches. A will show only the ASCII text after the packet line, instead of both ASCII and Hex. L will tell tcpdump to output packets in a different mode. L will line buffer instead of pooling and pushing in chunks. It allows us to send the output directly to another tool such as grep using a pipe |.

### Tips and Tricks

```shellsession
@htb[/htb]$sudo tcpdump -Ar telnet.pcap
```

<img width="656" height="575" alt="image" src="https://github.com/user-attachments/assets/b0a487c9-37a1-49f7-8307-a1a9abec9130" />

Notice how it has the ASCII values shown below each output line because of our use of -A. This can be helpful when quickly looking for something human-readable in the output.

### Piping a Capture to Grep

```shellsession
@htb[/htb]$ sudo tcpdump -Ar http.cap -l | grep 'mailto:*'
```

<img width="622" height="184" alt="image" src="https://github.com/user-attachments/assets/84673dbd-f073-415f-b07e-69403f4bf6fd" />

Using -l in this way allowed us to examine the capture quickly and grep for keywords or formatting we suspected could be there. In this case, we used the -l to pass the output to grep and looking for any instance of the phrase mailto:*. This shows us every line with our search in it, and we can see the results above. Using modifiers and redirecting output can be a quick way to scrape websites for email addresses, naming standards, and much more.

We can dig as deep as we wish into the packets we captured. It requires a bit of knowledge of how the protocols are structured, however. For example, if we wanted to see only packets with the TCP SYN flag set, we could use the following command:

### Looking for TCP Protocol Flags

```shellsession
@htb[/htb]$ tcpdump -i eth0 'tcp[13] &2 != 0'
```

This is counting to the 13th byte in the structure and looking at the 2nd bit. If it is set to 1 or ON, the SYN flag is set.

### Hunting For a SYN Flag

```shellsession
@htb[/htb]$ sudo tcpdump -i eth0 'tcp[13] &2 != 0'
```

<img width="632" height="551" alt="image" src="https://github.com/user-attachments/assets/02cfd8a0-3303-4974-8dec-92e02f173d9a" />

Our results include only packets with the TCP SYN flag set from what we see above.

TCPDump can be a powerful tool if we understand our networking and how hosts interact with one another. Take the time to understand typical protocol header structures to spot the anomaly when the time comes. Here are a few links to further our studies on standard Protocols and their structures. Except for the Wikipedia link, each link should take us directly to the RFC that sets the standard in place for each.

### Protocol RFC Links

| Link | Description |
|---|---|
| IP Protocol | RFC 791 describes IP and its functionality. |
| ICMP Protocol | RFC 792 describes ICMP and its functionality. |
| TCP Protocol | RFC 793 describes the TCP protocol and how it functions. |
| UDP Protocol | RFC 768 describes UDP and how it operates. |
| RFC Quick Links | This Wikipedia article contains a large list of protocols tied to the RFC that explains their implementation. |
