<img width="650" height="234" alt="image" src="https://github.com/user-attachments/assets/20c23109-ba20-4d70-a62b-9e3213244383" /># ICMP Tunneling

**Related PCAP File(s):**

- `icmp_tunneling.pcapng`

Tunneling is a technique employed by adversaries in order to exfiltrate data from one location to another. There are many different kinds of tunneling, and each different kind uses a different protocol. Commonly, attackers may utilize proxies to bypass our network controls, or protocols that our systems and controls allow.

## Basics of Tunneling

Essentially, when an attacker wants to communicate data to another host, they may employ tunneling. In many cases, we might notice this through the attacker possessing some command and control over one of our machines. As noted, tunneling can be conducted in many different ways. One of the more common types is SSH tunneling. However, proxy-based, HTTP, HTTPs, DNS, and other types can be observed in similar ways.

<img width="900" height="375" alt="image" src="https://github.com/user-attachments/assets/df1dbc6d-65e3-4780-b6ad-36c541a82cde" />

The idea behind tunneling is that an attacker will be able to expand their command and control and bypass our network controls through the protocol of their choosing.

## ICMP Tunneling

In the case of ICMP tunneling an attacker will append data they want to exfiltrate to the outside world or another host in the data field in an ICMP request. This is done with the intention to hide this data among a common protocol type like ICMP, and hopefully get lost within our network traffic.

<img width="876" height="403" alt="image" src="https://github.com/user-attachments/assets/18baeffc-3092-4907-97ee-92d4b0fa15cf" />

## Finding ICMP Tunneling

Since ICMP tunneling is primarily done through an attacker adding data into the data field for ICMP, we can find it by looking at the contents of data per request and reply.

<img width="1167" height="359" alt="image" src="https://github.com/user-attachments/assets/2de31ba2-718d-4e38-ac27-9abf5d7389b4" />

We can filter our wireshark capture to only ICMP requests and replies by entering `ICMP` into the filter bar.

<img width="1088" height="276" alt="image" src="https://github.com/user-attachments/assets/74f74e54-d784-4145-9975-dfef101f0b0a" />

Suppose we noticed fragmentation occurring within our ICMP traffic as it is above, this would indicate a large amount of data being transferred via ICMP. In order to understand this behavior, we should look at a normal ICMP request. We may note that the data is something reasonable like 48 bytes.

<img width="658" height="262" alt="image" src="https://github.com/user-attachments/assets/17737943-c0d1-4108-a4b3-090e37a98c96" />

However a suspicious ICMP request might have a large data length like 38000 bytes.

<img width="650" height="234" alt="image" src="https://github.com/user-attachments/assets/6c831702-e76b-4a09-912a-c33cd38d26e6" />

If we would like to take a look at the data in transit, we can look on the right side of our screen in Wireshark. In this case, we might notice something like a Username and Password being pinged to an external or internal host. This is a direct indication of ICMP tunneling.

<img width="534" height="557" alt="image" src="https://github.com/user-attachments/assets/b76d4c50-9867-4f43-b640-ff68119b25ae" />

On the other hand, more advanced adversaries will utilize encoding or encryption when transmitting exfiltrated data, even in the case of ICMP tunneling. Suppose we noticed the following.

<img width="553" height="561" alt="image" src="https://github.com/user-attachments/assets/fa64da0f-74be-4bd2-879b-878147d6bc6d" />

We could copy this value out of Wireshark and decode it within linux with the base64 utility.

```shell
@htb[/htb]$ echo 'VGhpcyBpcyBhIHNlY3VyZSBrZXk6IEtleTEyMzQ1Njc4OQo=' | base64 -d
```

This would also be a case where ICMP tunneling is observed. In many cases, if the ICMP data length is larger than 48-bytes, we know something fishy is going on, and should always look into it.

## Preventing ICMP Tunneling

In order to prevent ICMP tunneling from occurring we can conduct the following actions.

- **Block ICMP Requests** - Simply, if ICMP is not allowed, attackers will not be able to utilize it.
- **Inspect ICMP Requests and Replies for Data** - Stripping data, or inspecting data for malicious content on these requests and replies can allow us better insight into our environment, and the ability to prevent this data exfiltration.

## 🔑 Key Takeaway

ICMP tunneling allows attackers to hide exfiltrated data inside ICMP request and reply packets to bypass network controls. Monitoring ICMP payload sizes, fragmentation, and packet contents can help identify suspicious activity. Blocking unnecessary ICMP traffic and inspecting ICMP data can help prevent this form of data exfiltration.
