# HTTP/HTTPs Service Enumeration

**Related PCAP File(s):**

- `basic_fuzzing.pcapng`

Many times, we might notice strange traffic to our web servers. In one of these cases, we might see that one host is generating excessive traffic with HTTP or HTTPs. Attackers like to abuse the transport layer many times, as the applications running on our servers might be vulnerable to different attacks. As such, we need to understand how to recognize the steps an attacker will take to gather information, exploit, and abuse our web servers.

Generally speaking, we can detect and identify fuzzing attempts through the following:

- Excessive HTTP/HTTPs traffic from one host
- Referencing our web server's access logs for the same behavior

Primarily, attackers will attempt to fuzz our server to gather information before attempting to launch an attack. We might already have a Web Application Firewall in place to prevent this, however, in some cases we might not, especially if this server is internal.

## Finding Directory Fuzzing

Directory fuzzing is used by attackers to find all possible web pages and locations in our web applications. We can find this during our traffic analysis by limiting our Wireshark view to only http traffic.

```text
http
```

<img width="895" height="415" alt="image" src="https://github.com/user-attachments/assets/0183559a-777c-4864-a615-0aaf42cf5f00" />

Secondarily, if we wanted to remove the responses from our server, we could simply specify `http.request`

<img width="837" height="363" alt="image" src="https://github.com/user-attachments/assets/5d43b935-1a59-4495-90ba-d833ff5a81b8" />

Directory fuzzing is quite simple to detect, as it will in most cases show the following signs:

- A host will repeatedly attempt to access files on our web server which do not exist (response 404).
- A host will send these in rapid succession.

We can also always reference this traffic within our access logs on our web server. For Apache this would look like the following two examples. To use grep, we could filter like so:

```shell
@htb[/htb]$ cat access.log | grep "192.168.10.5"
```

<img width="761" height="505" alt="image" src="https://github.com/user-attachments/assets/97318b2d-ac68-405e-a979-1a7f23e087f6" />

And to use awk, we could do the following:

```shell
@htb[/htb]$ cat access.log | awk '$1 == "192.168.10.5"'
```

<img width="769" height="651" alt="image" src="https://github.com/user-attachments/assets/838c4555-690a-43a3-92fa-7bdf23989e1b" />

## Finding Other Fuzzing Techniques

However, there are other types of fuzzing which attackers might employ against our web servers. Some of these could include fuzzing dynamic or static elements of our web pages such as id fields. Or in some other cases, the attacker might look for IDOR vulnerabilities in our site, especially if we are handling json parsing (changing return=max to return=min).

To limit traffic to just one host we can employ the following filter:

```text
http.request and ((ip.src_host == <suspected IP>) or (ip.dst_host == <suspected IP>))
```

<img width="872" height="411" alt="image" src="https://github.com/user-attachments/assets/be9702a1-6658-40cb-bd91-bf770264cad2" />

Secondarily, we can always build an overall picture by right clicking any of these requests, going to follow, and follow HTTP stream.

<img width="586" height="635" alt="image" src="https://github.com/user-attachments/assets/f5f744ea-ff4e-40d3-9cf3-5343f2c35be8" />

Suppose we notice that a lot of requests were sent in rapid succession, this would indicate a fuzzing attempt, and we should carry out additional investigative efforts against the host in question.

However sometimes attackers will do the following to prevent detection:

- Stagger these responses across a longer period of time.
- Send these responses from multiple hosts or source addresses.

## Preventing Fuzzing Attempts

We can aim to prevent fuzzing attempts from adversaries by conducting the following actions.

- Maintain our virtualhost or web access configurations to return the proper response codes to throw off these scanners.
- Establish rules to prohibit these IP addresses from accessing our server through our web application firewall.

## 🔑 Key Takeaway

HTTP/HTTPs fuzzing involves sending repeated requests to discover directories, files, parameters, and potential vulnerabilities in web applications. Detecting excessive requests, repeated 404 responses, suspicious request patterns, and unusual activity in web server access logs can help identify fuzzing attempts. Web access configurations and Web Application Firewall rules can help prevent or limit this activity.
